# Security maintenance, 13 October 2026

The two Node HTTP services have updated dependencies and clean Yarn advisory scans. This is a partial repository remediation, not a security clearance for the mobile client, embed project, or Go server.

| Project | Before: advisory paths | After: advisory paths | Validation |
| --- | ---: | ---: | --- |
| bskylink | 22 (1 critical, 8 high) | 0 | `yarn build`, `yarn audit --json` |
| bskyogcard | 19 (1 critical, 4 high) | 0 | `yarn build`, `yarn audit --json` |
| Repository root | 1,496 (73 critical, 795 high) | Unchanged | Audit only |
| bskyembed | 131 (79 high) | Unchanged | Audit only |

Counts are dependency paths reported by Yarn, so a vulnerability may be counted repeatedly. Audit databases change over time and zero known advisories does not establish the absence of application vulnerabilities.

Express remains on major version 4. Updated Express, body-parser and their dependencies address request parsing, routing and redirect advisories. Kysely moves from 0.27 to 0.28.17, the first version fixing its JSON path traversal advisory; the existing service compiles with the new API. Narrow resolutions ensure patched proxy-addr, fflate and diff versions remain selected when the lockfile is regenerated. Dependencies were installed with lifecycle scripts disabled.

Relevant upstream advisories:

- [proxy-addr trust subnet IP spoofing](https://github.com/advisories/GHSA-jqcg-44mw-7w3h), requires a vulnerable trust subnet configuration.
- [Kysely JSON path traversal injection](https://github.com/advisories/GHSA-pv5w-4p9q-p3v2), applies when attacker controlled path legs reach the affected query API.
- [body-parser request parsing denial of service](https://github.com/advisories/GHSA-qwcr-r2fm-qrc7), applies to affected URL encoded parsing.

The PostgreSQL integration tests and image rendering service were not exercised against live dependencies. Compilation passed for both services. No deployment or database migration was performed.

A root dependency refresh failed during package download because the host ran out of disk space, before changing the root lockfile. Its critical findings include form-data, fast-xml-parser, handlebars, tar, shell-quote, websocket-driver, proxy-addr, elliptic and Babel dependencies. The mobile project needs a separately validated update of its Expo/React Native toolchain, including patch-package compatibility and native/web builds. The embed project needs a supported Vite/tooling migration; known advisories are not all fixed on Vite 5. Go tooling was unavailable, so bskyweb/go.mod and go.sum need a govulncheck run and runtime updates. None of these remaining areas is represented as fixed by this change.
