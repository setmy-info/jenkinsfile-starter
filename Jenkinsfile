
def runCommand(String command) {
    if (isUnix()) {
        sh command
    } else {
        bat command
    }
}

pipeline {

    /*
    version 1.1.0 - hotfix* branch support (Publish/Hotfix candidate stage, HOTFIX_TO_* deploy flags), TEST -> TEST (ADR-0041 canonical environment name), dead
    version 1.0.1 - fileExists precondition check now actually gates (was a discarded boolean)

    Git branches flow: develop -> feature -> develop -> release -> master

    Steps
    1. Enhancement event
    2. feature branch from develop
    3. Enhancements in feature branch - constant build in Jenkins by commits, cancel/avoid previous un started commits, try to execute only last
    4. After successful build merge do develop - constant build in Jenkins by commits, cancel/avoid previous un started commits, try to execute only last
    5. Go-No go event: Positive release and release testing decision by DEV and TEST environments
    6. Make release branch - constant build in Jenkins by commits, cancel/avoid previous un started commits, try to execute only last. Code freeze period started.
    7. Go-No go event: Positive release decision by DEV, TEST, PRELIVE environments
    8. Merge release branch to master - constant build in Jenkins by commits, cancel/avoid previous un started commits, try to execute only last. Code freeze period ended.
    9. Found a bug in production
    10. hotfix branch from master
    11. Enhancements in hotfix branch - constant build in Jenkins by commits, cancel/avoid previous un started commits, try to execute only last
    12. Go-No go event: Positive release decision by TEST, PRELIVE environments. Decisions to do merging in steps or not (go to development testing, test testing and then decide again - desisions about next steps usage).
    13. Hotfix merged develop.
    14. Some previous steps activated.
    15. Hotfix merged to master.

    Automatic deployments to environments.

    No pull request builds.

    [5 branches] x [4 environments] x [ 2 types: Automatic, Manual]
    */

    agent any

    triggers {
        cron('H/5 * * * *')
    }

    options {
        buildDiscarder(
            logRotator(
                numToKeepStr: '20',
                artifactNumToKeepStr: '10'
            )
        )
        //disableConcurrentBuilds(abortPrevious: true)
    }

    environment {
        //Just for example
        //PATH = "/opt/setmy.info/bin:$PATH"
        ABC = 'DEF'
        GHI = "$ABC"

        // hotfix* - branched from master, one fix, quick review + the FULL
        // automated test path (nothing is skipped), merged to master, which
        // then deploys live and tags. A hotfix reaches the same
        // pre-production targets a release does and never goes live directly.

        MASTER_TO_LIVE = 'DEPLOY'

        //MASTER_TO_PRELIVE = 'DEPLOY'
        RELEASE_TO_PRELIVE = 'DEPLOY'
        HOTFIX_TO_PRELIVE = 'DEPLOY'

        DEVELOPMENT_TO_TEST = 'DEPLOY'
        RELEASE_TO_TEST = 'DEPLOY'
        HOTFIX_TO_TEST = 'DEPLOY'

        DEVELOPMENT_TO_DEV = 'DEPLOY'
        RELEASE_TO_DEV = 'DEPLOY'
        //HOTFIX_TO_DEV = 'SKIP'
    }

    stages {
        stage('Inspection') {
            parallel {
                stage('Pre-build') {
                    steps {
                        echo "Jenkins node: ${env.NODE_NAME}"
                        echo "Operating system: ${isUnix() ? 'Unix/Linux' : 'Windows'}"

                        runCommand 'echo "Hello from command shell"'

                        script {
                            if (!fileExists('README.md')) {
                                error('README.md missing')
                            }
                        }

                        echo 'Pre build inspection and precondition check. Put here commands to check, that build tools are installed.'

                        runCommand 'echo "GHI=${GHI}"'
                        echo "Message GHI=${GHI}"

                        sleep 5

                        retry(count: 7) {
                            runCommand 'echo "Many times, why?"'
                        }

                        timeout(time: 10) {
                            runCommand 'echo "What is this time?"'
                        }

                        emailext(
                            subject: "Jenkins job: $JOB_NAME, build: $BUILD_NUMBER",
                            body: "Job: $JOB_NAME, build: $BUILD_NUMBER, url: ${env.BUILD_URL}",
                            recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                        )
                    }
                }
                stage('Build tools') {
                    steps {
                        echo 'Build tools installation and preparation (setup, config)'
                        runCommand 'echo "Hello stage B"'
                    }
                }
            }
        }

        stage('Preparation') {
            parallel {
                stage('Install') {
                    steps {
                        echo 'Preparing the software to be built. Installation commands go here.'
                        echo 'npm install'
                        echo 'Put here build configuration commands'
                        echo './config'
                    }
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Cleaning command, because in some cases shared directories can have previous build garbage'
                echo 'mvn clean'

                echo 'Put here resource copy commands'
                echo 'command to prepare files'

                echo 'Put here compilation commands. Can be omitted.'
                echo 'mvn compile -Pci'
                echo 'mvn test-compile -Pci'

                echo 'Put here unit tests'
                echo 'mvn test -Pci'

                echo 'Put here integration tests. Previous steps can be merged here,.'
                echo 'mvn verify -Pci'
                echo 'Or just: mvn clean verify -Pci (without previous)'

                echo 'Put here mutation tests'
                echo 'mvn org.pitest:pitest-maven:mutationCoverage'

                echo 'Put here reporting builds steps can include (unit tests coverage, mutation test coverage, findbugs, vuln. checks, )'
                echo 'Containing here findbug/stopbug, check style, dependencies vulnreability checks, docs gen, etc'
                echo 'mvn site:site -Pci'

                echo 'Put here site deploy'
                echo 'mvn site:deploy -Pci'

                echo 'Put here e2e tests'
                echo 'mvn verify -Pe2e -Pci'

                echo 'Put here system tests'
                echo 'Put here acceptance tests'

                echo 'Put here packaging'
                echo 'mvn package -DskipTests -DskipITs -Pci'

                echo 'Put here local publishing'
                echo 'mvn deploy -DskipTests -DskipITs -Pci'
            }
        }

        stage('Publish') {
            parallel {
                stage('Release') {
                    when {
                        branch 'master'
                        // changeset "**/file/to/be/changed"
                    }
                    steps {
                        echo 'Put here software release steps'
                    }
                }
                stage('Snapshot') {
                    when {
                        expression { env.BRANCH_NAME.startsWith('devel') }
                    }
                    steps {
                        echo 'Put here software snapshot publishing steps'
                    }
                }
                stage('Hotfix candidate') {
                    when {
                        expression { env.BRANCH_NAME.startsWith('hotfix') }
                    }
                    steps {
                        echo 'Put here software hotfix-candidate publishing steps'
                    }
                }
                stage('Release reports') {
                    when {
                        branch 'master'
                    }
                    steps {
                        echo 'Put here reports publishing steps'
                    }
                }
                stage('Snapshot reports') {
                    when {
                        expression { env.BRANCH_NAME.startsWith('devel') }
                    }
                    steps {
                        echo 'Put here reports publishing steps'
                    }
                }
            }
        }
        stage('Deploy') {
            parallel {
                stage('dev') {
                    when {
                        expression {
                            (env.DEVELOPMENT_TO_DEV == 'DEPLOY' && env.BRANCH_NAME.startsWith('devel')) ||
                            (env.RELEASE_TO_DEV == 'DEPLOY' && env.BRANCH_NAME.startsWith('release'))
                        }
                    }
                    steps {
                        echo 'Put here software development installations steps'
                    }
                }
                stage('test') {
                    when {
                        expression {
                            (env.DEVELOPMENT_TO_TEST == 'DEPLOY' && env.BRANCH_NAME.startsWith('devel')) ||
                            (env.RELEASE_TO_TEST == 'DEPLOY' && env.BRANCH_NAME.startsWith('release')) ||
                            (env.HOTFIX_TO_TEST == 'DEPLOY' && env.BRANCH_NAME.startsWith('hotfix'))
                        }
                    }
                    steps {
                        echo 'Put here software development installations steps'
                    }
                }
                stage('prelive') {
                    when {
                        expression {
                            (env.RELEASE_TO_PRELIVE == 'DEPLOY' && env.BRANCH_NAME.startsWith('release')) ||
                            (env.HOTFIX_TO_PRELIVE == 'DEPLOY' && env.BRANCH_NAME.startsWith('hotfix'))
                        }
                    }
                    steps {
                        echo 'Put here software prelive installations steps'
                    }
                }
                stage('live') {
                    when {
                        expression {
                            env.MASTER_TO_LIVE == 'DEPLOY' && env.BRANCH_NAME == 'master'
                        }
                    }
                    steps {
                        echo 'Put here software production installations steps'
                    }
                }
            }
        }
        stage('Tag') {
            when {
                branch 'master'
                expression { env.MASTER_TO_LIVE == 'DEPLOY' }
            }
            steps {
                echo 'Put here taging'
            }
        }
    }

    post {
        always {
            // junit '**/target/*-reports/*.xml'
            runCommand 'echo "Allways"'
        }

        success {
            emailext (
                subject: "Jenkins job: $JOB_NAME, build: $BUILD_NUMBER type: SUCCESSFUL",
                body: "Job: $JOB_NAME, build: $BUILD_NUMBER, url: ${env.BUILD_URL}, git: ${env.GIT_URL}, branch: ${env.GIT_BRANCH} SUCCESSFUL post step",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
        }

        failure {
            emailext (
                subject: "Jenkins job: $JOB_NAME, build: $BUILD_NUMBER type: FAILED",
                body: "Job: $JOB_NAME, build: $BUILD_NUMBER, url: ${env.BUILD_URL}, git: ${env.GIT_URL}, branch: ${env.GIT_BRANCH}  FAILED post step",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
        }
    }
}
