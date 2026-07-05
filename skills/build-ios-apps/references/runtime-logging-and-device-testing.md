# Runtime Logging And Device Testing

Read this when adding diagnostics, debugging a physical-device report, or making an iOS app testable through Simulator and a connected iPhone.

## Logging Goals

- Add enough detail to diagnose failures after the user tests on a physical iPhone, but keep logs compact enough to read in context.
- Use `os.Logger` for system logs and a small JSON Lines file sink for app-owned logs that can be exported or copied from the app container.
- Keep one event per line. Use stable event names and short fields; record counts, IDs, state names, and error codes instead of dumping full objects.
- Redact tokens, secrets, personal content, full request/response bodies, and precise user data. Truncate long strings.
- Rotate logs. A practical default is 256 KB per file, 3 files retained, debug/development builds only unless the product needs production diagnostics.

## Event Shape

Use a shape like this for the file sink:

```json
{"ts":"2026-07-05T12:34:56Z","level":"info","area":"sync","event":"load.start","screen":"Feed","run":"A1B2","fields":{"count":42}}
{"ts":"2026-07-05T12:34:57Z","level":"error","area":"network","event":"request.failed","screen":"Feed","run":"A1B2","error":{"domain":"NSURLErrorDomain","code":-1009,"message":"offline"}}
```

Useful fields:

- `ts`, `level`, `area`, `event`, `screen`, `run`
- app version/build, device model, OS version, locale, Dynamic Type size, color scheme at launch
- feature flags, data fixture name, permission state, network state, request ID
- database migration names, cache hits/misses, empty/error/loading state transitions
- error domain/code/message and a short call-site or operation name

## Where To Store Logs

- Put app-owned logs under Application Support or Documents, for example `Application Support/Diagnostics/runtime.jsonl`.
- Add a debug-only in-app export/share action when physical-device retrieval is otherwise awkward.
- For UI tests, enable diagnostics and fixture selection through launch arguments or environment variables such as `UITEST_FIXTURE=large` and `DIAGNOSTICS_ENABLED=1`.
- Keep the logging helper small and local, for example `Diagnostics/AppDiagnostics.swift`; do not add a large logging framework for simple app work.

## Reading Logs

Simulator app container:

```sh
xcrun simctl get_app_container booted <bundle-id> data
find <container-path> -name '*runtime*.jsonl' -o -name '*diagnostic*.jsonl'
```

Simulator system log stream:

```sh
xcrun simctl spawn booted log stream --style compact --predicate 'subsystem == "<bundle-id>"'
```

Physical iPhone:

- Prefer the app's debug export/share action when present.
- Use Xcode Devices and Simulators to download the app container when available.
- Check current `devicectl` file/log support before relying on exact commands:
  ```sh
  xcrun devicectl --help
  xcrun devicectl device --help
  xcrun devicectl device info files --help
  xcrun devicectl device copy --help
  ```
- If command-line file extraction is unsupported by the installed Xcode/device combination, state that and use the debug export path or Xcode UI.

## Test Data

Create deterministic fixtures that cover:

- empty, one item, typical list, and very large list
- long titles, long body text, missing optional fields, duplicate names, unusual Unicode if the app supports user text
- loading, offline, timeout, server error, permission denied, and expired session states
- first launch, returning launch, migration/upgrade, logout/reset
- portrait and landscape, light and dark mode, large Dynamic Type, reduced motion, high contrast

Name fixtures clearly so logs and screenshots explain the state, for example `fixture=large-offline-dark`.
