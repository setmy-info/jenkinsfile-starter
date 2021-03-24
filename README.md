# jenkinsfile-starter
Jenkinsfile example and probe project to run in Jenkins.

## Requirements

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
