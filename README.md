# jenkinsfile-starter
Jenkinsfile example and probe project to run in Jenkins.

## Requirements

Master should go to live. No any other branch should go to live. Mostly conditional switch should be switched on.

Release branch can go dev, test, prelive by conditional swirches.

Develpoment branch is optional. Mostly Jenkisnfile is planned for trunk based development scenarios.

Hotfix branch is not supported, because its can be and should be handled as release branch.

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
