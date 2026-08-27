# jenkinsfile-starter

Jenkinsfile example and probe project to run in Jenkins.

## Goals

To get faster feedback, why is failing, in order:

* code doesn't compile
* test code doesn't compile
* unit tests are failing
* integration tests are failing
* mutation tests are failing
* qualiti measurements are failing
* e2e tests are failing
* acceptance tests are failing

## Requirements

Master should go to live. No any other branch should go to live. Mostly conditional switch should be switched on.

Release branch can go dev, test, prelive by conditional swirches.

Develpoment branch is optional. Mostly Jenkisnfile is planned for trunk based development scenarios.

Hotfix branch (`hotfix*`, branched from master) is supported since 1.1.0: it runs the full build/test path like every
branch, publishes a hotfix candidate, and can deploy to test and prelive by conditional switches (`HOTFIX_TO_TEST`,
`HOTFIX_TO_PRELIVE`; `HOTFIX_TO_DEV` off by default). It never goes live itself - after quick review and test it is
merged to master, and the master build deploys live and tags.

### Publishing by branch

| Branch     | Release | Snapshot | Release reports | Snapshot reports |
|------------|:-------:|:--------:|:---------------:|:----------------:|
| `feature*` |    –    |    –     |        –        |        –         |
| `develop`  |    –    |    ✓    |        –        |        ✓        |
| `release*` |    –    |    –     |        –        |        –         |
| `hotfix*`  |    –    |    –     |        –        |        –         |
| `master`   |   ✓    |    –     |       ✓        |        –         |

### Deployment by branch and environment

| Branch     | DEV | TEST | PRELIVE | LIVE |
|------------|:---:|:----:|:-------:|:----:|
| `feature*` |  –  |  –   |    –    |  –   |
| `develop`  | ✓  |  ✓  |    –    |  –   |
| `release*` |  –  |  ✓  |   ✓    |  –   |
| `hotfix*`  |  –  |  ✓  |   ✓    |  –   |
| `master`   |  –  |  –   |    –    |  ✓  |

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
