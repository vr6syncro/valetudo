# Xiaomi Mi Robot Vacuum Gen1: Patched Valetudo RE ARMv7

This fork preserves Valetudo RE's upstream `testing` branch and adds:
- A guard for malformed/missing `get_status` results in `lib/miio/Dummycloud.js`. An invalid status response is logged and ignored instead of causing an uncaught `TypeError`.
- Small regression tests for STATUS response validation (`npm test`).
- A GitHub Actions build of the existing `node16-linuxstatic-armv7` target. It verifies the unit tests, German and Traditional Chinese JSON locales, and ARM ELF headers; uploads a binary and SHA256 checksum.
- Safe, manually ported translations from upstream [#450](https://github.com/rand256/valetudo/pull/450) and [#544](https://github.com/rand256/valetudo/pull/544), and a `config.json` ignore rule from [#110](https://github.com/rand256/valetudo/pull/110). See [UPSTREAM-PR-REVIEW.md](UPSTREAM-PR-REVIEW.md).

**This is an application-only update**, not a replacement firmware image. It has been tested on an original Xiaomi V1 running vendor firmware `3.5.8_004028` and Linux 3.4.39.

## Binary versioning

The source retains the historical upstream `package.json` version `0.10.0`. During GitHub Actions builds **only**, the workflow inserts an unambiguous suffix such as:

`0.10.0+vr6syncro.e4d26e387cff`

That suffix identifies the commit used for the artifact. **It is not an upstream release or a claim of version 0.10.11**. The original installed patched build without the suffix displayed `0.10.0`.

## Build and verification

Open the [GitHub Actions workflow](../../actions/workflows/build-armv7.yml), select a successful run and download its `valetudo-re-v1-armv7-<SHA>` artifact.

The artifact contains:
- `valetudo` – standalone 32-bit ARM/EABI5 binary, built using `pkg`;
- `SHA256SUMS` – SHA256 of the actual executable;
- `BUILD-INFO.txt` – GitHub commit, ref, run ID and displayed version;
- `FILE-INFO.txt`, `ELF-HEADER.txt`, `ELF-PROGRAM-HEADERS.txt` – binary inspection output.

On a local Windows machine with GitHub CLI, for example:

```powershell
gh run download <run-id> --repo vr6syncro/valetudo --name <artifact-name> --dir .\valetudo-armv7
Get-FileHash .\valetudo-armv7\valetudo -Algorithm SHA256
Get-Content .\valetudo-armv7\SHA256SUMS
```

Do **not** use an unverified executable or overwrite the robot's currently running binary during a cleaning cycle.

## Safe deployment outline

1. Confirm the device is idle, adequately charged, and SSH works. Confirm its actual Upstart service, binary location, permissions, and mount layout.
2. Back up the existing binary and `/etc/init/valetudo.conf` along with the relevant configuration locally. Capture its SHA256, permissions and ownership.
3. Transfer the verified artifact to a temporary location in `/mnt/data`. Re-check SHA256 on the device before use.
4. Prepare a reversible replacement and prove the backup can be restored. Stop the Valetudo service via Upstart; replace only the executable at the confirmed installation path, retaining correct mode/ownership; restart the service.
5. Allow adequate startup time before judging the map and API readiness. The UI may respond before the map is populated. Verify SSH, HTTP, status, map, logs, PID, memory and robot controller stability.
6. Monitor a normal cleaning run, including vendor navigation log lines, and document any malformed STATUS warnings. Never inject packets or disable safety sensors for this test.
7. If the new executable fails, stop the service, restore the backed-up executable and restart it. Do not reset the robot or reflash partitions.

For device logs in the tested Ubuntu 14.04/Upstart environment: `/var/log/upstart/valetudo.log`.

## Field test — 2026-10-11

An integration build of this fork was deployed on an actual Xiaomi Mi Robot Vacuum Gen1.

- Original vendor firmware: `3.5.8_004028`; Ubuntu 14.04.3, kernel `3.4.39`.
- Tested build: [Actions run #38141788249](https://github.com/vr6syncro/valetudo/actions/runs/38141788249), commit `9cb40353799d69cb875cdea81ca713fe1763fbf0`.
- Executable SHA256: `f2b073a8cff4fdfc958f8cc457e809e03f05e2c4a115861d657403fbdd671736`.
- SSH, HTTP web UI (HTTP 200), robot status and map (6,242 bytes) worked after startup. Configuration was retained; rollback was successfully exercised.
- Short observation: no new JavaScript exceptions reported; patched process was stable, with RSS 35,488 KiB versus 36,540 KiB previously.
- Later user feedback: robot **cleaned approximately the full intended area**; the user **manually ordered return to the charging station**.

**Limitations:** The reported cleaning run was not a verified autonomous `clean_complete`; duration, area, final error code and the full 20-minute monitoring result were not provided. The original malformed STATUS error was *not* reproduced against the patched binary in live operation. No claim is made that the separate intermittent `BlockCrosser: No entrance!` vendor navigation problem is fixed.

A subsequent successful workflow run generated a **new, uniquely stamped binary**, which was **not** installed for this test; do not describe that later build as field-tested.

## Upstream contribution

For submission to original author `rand256`, keep the STATUS parser fix and tests separate from localization and CI. The branch [`fix/dummycloud-status-crash`](../../tree/fix/dummycloud-status-crash) contains the minimal fix. Discussions and device test evidence are recorded in the fork's [PR #1](../../pull/1).

Note that upstream PR [#451](https://github.com/rand256/valetudo/issues/451) concerns early cleaning completion resolved by cleaning a side sensor on another device; it is not proof of the cause on this V1.
