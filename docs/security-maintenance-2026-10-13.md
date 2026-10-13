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


## Service container runtime followup

Both build and runtime stages in `Dockerfile.bskylink` and `Dockerfile.bskyogcard` now use the official `node:22.23.3-alpine3.24` image. The previous Node 20 release line reached end of support on 30 April 2026. Node 22 remains in maintenance support until 30 April 2027, according to the [Node release schedule](https://github.com/nodejs/Release/blob/main/schedule.json).

The exact tag was verified against the [official image manifest](https://github.com/docker-library/official-images/blob/master/library/node) and with `docker manifest inspect`, including Linux AMD64 and ARM64 variants. The [upstream image definition](https://github.com/nodejs/docker-node/blob/main/22/alpine3.24/Dockerfile) includes Yarn 1.22.22, matching this repository's package manager. Official npm engine metadata for the service runtime, native renderer and compiler dependencies accepts Node 22.23.3, including Kysely's Node 20 minimum and resvg's Node 10 minimum.

No image layers were downloaded and no container build or container runtime test was performed because available disk space was insufficient for another dependency installation. The earlier successful TypeScript builds used host Node 26.8.1. Container validation, including the native renderer and font download step, remains required before deploying these images. The mobile and embed Dockerfiles were not changed.
