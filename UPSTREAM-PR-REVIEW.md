# Review of open upstream pull requests (2026-10-11)

This branch is an **integration build** for Xiaomi Mi Robot Vacuum Gen1, based on `build/v1-armv7`. It includes the isolated Dummycloud STATUS-response crash fix, tests, and the fork-only GitHub Actions ARMv7 builder.

## Incorporated changes

| Upstream PR | Source | Changes | Outcome |
|---|---|---|---|
| [#450](https://github.com/rand256/valetudo/pull/450) | `client/locales/de.json` | Minor German UI wording and punctuation | Incorporated |
| [#544](https://github.com/rand256/valetudo/pull/544) | `client/locales/tw.json` | Traditional Chinese UI localization improvements | Incorporated |
| [#110](https://github.com/rand256/valetudo/pull/110) | `.gitignore` | Ignore `config.json` | Incorporated |

These changes were **manually ported by matching source hunks**, not merged or marked as accepted in the original upstream repository. The original authors and PR links are retained for provenance.

## Not incorporated

- [#408 – Fix build](https://github.com/rand256/valetudo/pull/408): Targets an old Node 14 / `pkg 4.4.9` toolchain and is not directly applicable to the current `node16-linuxstatic-armv7` / `pkg 5.8.1` build; the GitHub Actions build is working without it.
- [#62 – Update rc.local](https://github.com/rand256/valetudo/pull/62): Blanket iptables DROP rules risk blocking existing and related SSH/HTTP traffic without robust connection tracking, as well as restricting robot connectivity. Requires a separate threat-modelled review and safe recovery plan.

## Validation and scope

- Translation JSON structure verified after porting.
- ARMv7 build and unit-test status: see GitHub Actions for this branch.
- Build output must **not** be installed on the robot without backing up the existing binary, checking the SHA256 and runtime compatibility, and preparing rollback.
- These UI translations **do not** change the robot's spoken voice language.
- The early vendor navigation `BlockCrosser: No entrance!` behavior is independent of the proven Valetudo STATUS parser crash, and remains under investigation.
