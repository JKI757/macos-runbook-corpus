# Hermes WebUI and Matrix Recovery After a macOS Update

## Scope

Recover a previously working Hermes installation after a source update changes Python dependency management. Verified with Hermes Agent 0.22.2 and the nesquena/hermes-webui bootstrap. Re-check current upstream behavior before applying version-specific repairs.

Distinct services:

- Hermes WebUI / Hermex-compatible server: typically port 8787.
- Hermes gateway OpenAI-compatible API: typically localhost port 8642.
- Matrix adapter: runs inside the gateway, with its own authentication and encryption state.
- Hermes dashboard and Open WebUI are different products; do not restart them merely because this WebUI is down.

A successful update receipt or launchd `state = running` does **not** establish service health.

## 1. Capture evidence before restarting

```sh
hermes --version
hermes pm status
hermes gateway status
lsof -nP -iTCP:8787 -sTCP:LISTEN
lsof -nP -iTCP:8642 -sTCP:LISTEN
curl -fsS --max-time 8 http://127.0.0.1:8787/health
curl -fsS --max-time 8 http://127.0.0.1:8642/health
```

Privately inspect gateway logs, WebUI stderr, the relevant LaunchAgent definitions, and watchdog records. Do not publish raw environment files, tokens, recovery keys, passwords, room IDs, account IDs, or full process environments.

Look for these independent failures:

| Evidence | Likely cause |
| --- | --- |
| WebUI bootstrap cannot import both WebUI and Hermes; `No module named yaml` after an interpreter re-exec | Managed interpreter selected without its committed dependency activation environment |
| Matrix extra unavailable with a Linux-only platform gate; no Matrix adapter | Matrix packages absent from the **active** managed environment |
| Matrix connected but API port absent | Legacy API configuration no longer loaded into the expected environment |
| Gateway repeatedly restarts at watchdog intervals | Watchdog API probe fails even though Matrix or the gateway process is alive |

## 2. Repair WebUI runtime activation

Do not hardcode an `installs/<id>/environments/<generation>/venv` path in the WebUI service. Generations can change or be collected. Installing into the historical `hermes-agent/venv` does not guarantee the updated process uses those packages.

The verified bootstrap repair resolves both the managed interpreter and the committed activation environment **before** testing imports:

1. Discover the Hermes source directory as normal.
2. In a stdlib-only subprocess, place that source directory on `sys.path`.
3. Import `resolve_store_python` from `hermes_cli._launchers` and `activation_environment` from `pm.environments`.
4. Return the interpreter and activation environment as JSON.
5. Apply the environment to the WebUI launcher, explicitly removing stale `PYTHONHOME` and `VIRTUAL_ENV`.
6. Only then run the existing combined `yaml` / `run_agent.AIAgent` import probe and launch the server with the resolved interpreter.
7. Retain the existing fallback when the checkout does not have PM environment support.

The companion [bootstrap patch](../patches/hermes-webui-managed-runtime.patch) records the exact implementation. It was tested against WebUI bootstrap blob `cf665fd8`; it may need adaptation on newer versions:

```sh
# In the local WebUI checkout; inspect existing changes first.
git status --short
git apply --check /path/to/hermes-webui-managed-runtime.patch
git apply /path/to/hermes-webui-managed-runtime.patch
git diff --check
```

This is a local compatibility patch, not an upstream fix. Preserve/review it before future WebUI updates. The PM functions are internal interfaces and may change. Do not merely suppress the startup import check: that can leave a healthy-looking UI whose chat backend cannot import Hermes.

Restart the existing WebUI LaunchAgent using its actual label. Use `kickstart -k` if loaded, or `bootstrap` its plist if absent; do not launch a second unmanaged copy. Restarting interrupts active WebUI work.

## 3. Restore Matrix dependencies in the actual environment

Determine the committed dependency environment from PM, not from an old shell's `VIRTUAL_ENV`. The WebUI discovery procedure above can also reveal the effective activation `PYTHONPATH`. `pm.environments.committed_venv(source_root)` identifies the committed venv.

Use that environment's Python for package installation. Prefer the supported Matrix extra when available. In the affected release it is Linux-gated, so macOS needs a manually built stack. A plain `hermes pm install` does not overcome that platform gate.

### macOS python-olm build repair

The verified Python 3.14 build used python-olm 3.2.16, with two upstream compatibility adjustments:

- Export `CMAKE_POLICY_VERSION_MINIMUM=3.5` for modern CMake.
- In the extracted source's `libolm/include/olm/list.hh`, change the single declaration `T * const other_pos` to `T * other_pos`; AppleClang rejects incrementing the const pointer.

Download the pinned source from the package index to a private build directory, safely extract it, inspect the change, and preserve the patched source or wheel in durable local dependency storage rather than auto-pruned scratch. Then:

```sh
# Set ACTIVE_PYTHON to the committed venv's bin/python, not the bare store Python.
# Set PATCHED_SOURCE to the inspected, patched python-olm-3.2.16 directory.
CMAKE_POLICY_VERSION_MINIMUM=3.5 uv pip install --python "$ACTIVE_PYTHON" \
  "$PATCHED_SOURCE" 'mautrix[encryption]==0.21.1' \
  asyncpg aiosqlite aiohttp-socks
```

Native build prerequisites include Xcode command-line tools and CMake. If using system libolm instead, verify its headers/library and package build instructions rather than assuming compiler flags disable the vendored build.

Verify the complete crypto import chain in the activated runtime:

```python
import mautrix, olm, asyncpg, aiosqlite
from mautrix.crypto import OlmMachine
from mautrix.crypto.store.asyncpg import PgCryptoStore
```

A future PM rebuild may replace this manually extended environment. Retain the source/wheel and reinstall into the newly committed environment if required; this repair is not a change to Hermes's dependency lock.

### Preserve Matrix identity

Do not delete the crypto store or rotate the device as an initial repair. Privately read the existing `device_id` from `platforms/matrix/store/crypto.db`, table `crypto_account`, and retain the same `MATRIX_DEVICE_ID` in the private `.env`. Password login without a stable ID can hit the homeserver device cap. Preserve existing password and recovery-key settings; do not disable encryption to claim encrypted rooms work.

## 4. Restore the gateway API configuration

If the gateway starts Matrix but not its API listener, check whether `API_SERVER_ENABLED` and `API_SERVER_KEY` exist only as legacy top-level config.yaml entries. In the verified repair, migrating those **existing values** into the default Hermes home's private `.env` restored the listener:

```text
API_SERVER_ENABLED=true
API_SERVER_KEY=<existing secret; do not replace or publish it>
```

Make a private backup first, do not create duplicate settings, preserve the existing key, and keep `.env` and backups mode 0600. Do not copy another profile's secrets. Retain a localhost bind unless exposure is explicitly intended and secured.

```sh
hermes gateway restart
curl -fsS --max-time 8 http://127.0.0.1:8642/health
```

Restarting the gateway can interrupt in-flight work. Allow shutdown/drain and startup to finish before judging results. Avoid repeated watchdog-driven restarts while diagnosing; correct the failed probe's underlying cause, then verify normal monitoring resumes.

## 5. Acceptance checks

### WebUI

- Intended listener is present and `/health` returns JSON `status: ok`.
- `/static/index.html` returns HTML, not merely HTTP 200 with an error page.
- `/api/auth/status` reports authentication enabled.
- Anonymous `/api/sessions` returns 401.
- Existing-password login returns 200 and issues a cookie; authenticated `/api/sessions` returns 200.
- Authenticated `/api/onboarding/status` reports `hermes_found`, `imports_ok`, `provider_ready`, and `chat_ready` as true.
- Repeat public probes on the intended LAN/Tailscale address. Host-local requests to that address do not establish reachability from a phone or another computer.
- Test a real browser chat separately if end-to-end model execution is required.

### Matrix and gateway

Require fresh logs from the latest start, not an old successful block:

- Logged in as the intended account.
- Cross-signing verified via recovery key, when configured.
- E2EE enabled with the preserved device ID.
- Initial sync complete and expected rooms joined.
- `matrix connected` and `api_server connected`.
- API `/health` responds on the intended localhost port.
- No ongoing invalid-token or decryption error loop.
- Fresh inbound/outbound encrypted messages are the end-to-end acceptance check; startup sync alone is not a delivered conversation.

Finally run the watchdog's gateway/WebUI checks: both should pass without requesting a restart. Keep unrelated service failures separate. Never report full recovery based only on process supervision or a successful package install.

## References

- [Hermes documentation](https://hermes-agent.nousresearch.com/docs/)
- [Hermes Agent source](https://github.com/NousResearch/hermes-agent)
- [Hermes WebUI source](https://github.com/nesquena/hermes-webui)
- [launchd User Service Recovery](launchd-user-service-recovery.md)
