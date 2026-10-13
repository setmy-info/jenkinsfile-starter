# jenkinsfile-starter

Jenkinsfile example and probe project to run in Jenkins.

## Goals

To get faster feedback, why is failing, in order:

* code doesn't compile
* test code doesn't compile
* unit tests are failing
* integration tests are failing
* mutation tests are failing
* quality measurements are failing
* e2e tests are failing
* acceptance tests are failing

## Requirements

Master should go to the live publish stage, draft to the live draft stage. No any other branch should go to live.
Mostly conditional switch should be switched on.

Release branch can go test, prelive by conditional switches.

Development branch is optional. Mostly Jenkinsfile is planned for trunk based development scenarios.

Hotfix branch (`hotfix*`, branched from master) is supported since 1.1.0: it runs the full build/test path like every
branch, publishes a hotfix candidate, and can deploy to test and prelive by conditional switches (`HOTFIX_TO_TEST`,
`HOTFIX_TO_PRELIVE`; `HOTFIX_TO_DEV` off by default). It never goes live itself - after quick review and test it is
merged to draft and draft to master, and the master build deploys live and tags.

Draft branch (`draft`) is supported since 1.3.0. Mostly for web, PWA, FE and CMS projects. Flow is
`release* -> draft -> master`, and releasing to LIVE goes in two stages:

* live draft stage (`DRAFT_TO_LIVE`) - LIVE, but seen only by reviewers. The draft build is the release: it builds,
  publishes the release package to package storage (Nexus, SCP, Samba, NFS, ... file server) and deploys it to the
  live draft stage.
* live publish stage (`MASTER_TO_LIVE`) - what everyone sees. Users accept the draft by merging `draft` to `master`
  (fast-forward). The master build still builds and tests, but publishes nothing: the release is already published.
  It promotes the same package from the live draft stage to the live publish stage and tags.

Releases are immutable: a published release version is never built or published again, what goes live is the package
that was reviewed. A rejected draft is fixed with a new version. How the package moves from the live draft stage to the
live publish stage is up to the project. Every change, hotfixes included, goes through `draft` before `master`. The
live draft stage holds one change: while it waits for acceptance, the next one waits behind it or is accepted together
with it.

TEST is set up as much as possible like LIVE, so that every aspect of LIVE can be tested there. It has the same two
stages, test draft stage and test publish stage. Every TEST deployment (`develop`, `release*`, `hotfix*`) goes to the
test draft stage, gated by the `*_TO_TEST` switches. The pipeline never deploys to the test publish stage: the draft
is published there manually or with other tools.

Software without drafting has no `draft` branch: the flow is `release* -> master`, and master releases and publishes,
as in 1.2.0. The build shifts back to master - Release and Release reports run on `master` instead of `draft`,
`DRAFT_TO_LIVE` is not used, and master publishes the release and deploys it to the live publish stage. In the tables
below the `draft` row then moves to `master`, and hotfixes are merged to master directly.

### Publishing by branch

| Branch     | Release | Snapshot | Release reports | Snapshot reports |
|------------|:-------:|:--------:|:---------------:|:----------------:|
| `feature*` |    –    |    –     |        –        |        –         |
| `develop`  |    –    |    ✓    |        –        |        ✓        |
| `release*` |    –    |    –     |        –        |        –         |
| `hotfix*`  |    –    |    –     |        –        |        –         |
| `draft`    |   ✓    |    –     |       ✓        |        –         |
| `master`   |    -    |    –     |        -        |        –         |

In software built without drafting, the `draft` branch is missing and `master` takes over the publishing that
`draft` does.

### Deployment by branch and environment

| Branch     | DEV | TEST draft stage | TEST publish stage | PRELIVE | LIVE draft stage | LIVE publish stage |
|------------|:---:|:----------------:|:------------------:|:-------:|:----------------:|:------------------:|
| `feature*` |  –  |        –         |         –          |    –    |        –         |         –          |
| `develop`  | ✓  |        ✓        |         –          |    –    |        –         |         –          |
| `release*` |  –  |        ✓        |         –          |   ✓    |        –         |         –          |
| `hotfix*`  |  –  |        ✓        |         –          |   ✓    |        –         |         –          |
| `draft`    |  –  |        –         |         –          |    –    |        ✓        |         –          |
| `master`   |  –  |        –         |         –          |    –    |        –         |         ✓         |

In software built without drafting, the `draft` branch is missing and `master` deploys directly to the LIVE publish
stage. There is no LIVE draft stage. In TEST draft will be published manually or with other tools.

### Jenkins plugins

Usually Jenkins preferred/default plugins setup.

1. Email Extension Plugin
2. Mailer Plugin

### Jenkins configuration

1. System Admin e-mail address
2. Extended E-mail Notification settings
3. E-mail Notification settings
4. Jenkins URL

### Tips and tricks

1. Environment variables and other variables usage

Note that " and ' have difference

``` groovy
ABC = 'DEF'
GHI = "$ABC"
```
