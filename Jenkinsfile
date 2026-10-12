
def runCommand(String command) {
    if (isUnix()) {
        sh command
    } else {
        bat command
    }
}

pipeline {

    /*
    version 1.3.0 - draft branch added. LIVE gets two stages: live draft stage (DRAFT_TO_LIVE)
                    and live publish stage (MASTER_TO_LIVE). The draft build is the release: it
                    publishes the release package to package storage and deploys it to the live
                    draft stage. Users accept it by merging draft to master. The master build
                    still builds and tests, but publishes nothing - it promotes the already
                    published package from the live draft stage to the live publish stage.
                    Releases are immutable: what goes live is the very package that was
                    reviewed, never a rebuild. Software without drafting keeps the 1.2.0 flow,
                    where master releases and publishes - see "without draft" below.
    version 1.2.0 - release* no longer deploys to DEV: the RELEASE_TO_DEV flag and the release
                    branch of the 'dev' deploy stage are gone. release* deploys to TEST and
                    PRELIVE only. develop -> DEV is unchanged.
    version 1.1.0 - pollSCM instead of cron (build on new commits, not on a timer),
                    quietPeriod + disableConcurrentBuilds(abortPrevious: true) so that a burst
                    of commits becomes one build of the newest change,
                    release* added to Publish/Snapshot: develop, release* and hotfix* all publish
                    a candidate of unknown quality
                    TEST environment renamed to the ADR-0041 canonical name
    version 1.0.1 - fileExists precondition check now actually gates (was a discarded boolean)

    Git branches flow: develop -> feature -> develop -> release -> draft -> master

    Building only the newest change
    Every branch is polled for new commits, and commits arrive in bursts. Building each one of
    them is wasted work, so a burst is collapsed twice: quietPeriod folds the commits that have
    not started building yet into a single build, and disableConcurrentBuilds(abortPrevious:
    true) aborts a run that is already in progress as soon as a newer one is ready. What gets
    built is the newest change; the intermediate ones are skipped, not queued. See the options
    block.

    Steps
    1. Enhancement event
    2. feature branch from develop
    3. Enhancements in feature branch - built on new commits, only the newest change is built
    4. After successful build merge to develop - built on new commits, only the newest change is built
    5. Go-No go event: Positive release and release testing decision by DEV and TEST environments
    6. Make release branch - built on new commits, only the newest change is built. Code freeze period started.
    7. Go-No go event: Positive release decision by DEV, TEST, PRELIVE environments
    8. Merge release branch to draft - built on new commits, the release package is published to
       package storage and deployed to the live draft stage. Users review it on the live system,
       nobody else sees it yet.
    9. Go-No go event: users accept the draft. Not accepted - fix in release with a new version
       (a published release is immutable) and go to step 8 again.
    10. Merge draft to master (fast-forward, so master is exactly the reviewed commit) - built and
        tested, the reviewed package is promoted from live draft stage to live publish stage, tagged. Code
        freeze period ended.
    11. Found a bug in production
    12. hotfix branch from master
    13. Enhancements in hotfix branch - built on new commits, only the newest change is built
    14. Go-No go event: the hotfix is deployed to TEST and PRELIVE, where QA validates and
        verifies it. The decision taken there is either "this is verified" - go to step 17 - or
        "this needs more testing" - go to step 15.
    15. Hotfix merged to develop, when the decision at step 14 asked for more testing.
    16. The development flow continues from step 4, so the fix now also reaches DEV and is
        tested again together with everything else on develop.
    17. Hotfix merged to draft, reviewed there, and draft merged to master (steps 8-10). master is
        what deploys live and tags.

    Automatic deployments to environments: every deployment below is triggered by the build
    itself and gated only by the *_TO_* flags in the environment block. There is no manual
    approval step (no input step) anywhere in this pipeline. The user acceptance of a draft is
    not an input step either: it is the merge of draft to master.

    draft - releasing to LIVE in two stages. The live draft stage is LIVE, only not visible to
    everyone yet: reviewers see it, users accept it there. The live publish stage is what
    everyone sees. Mostly for web, PWA, FE and CMS kind of projects. The draft build publishes
    the release package, once; from then on that package is only moved, never built again. How
    the package gets from the live draft stage to the live publish stage (install it again from
    package storage, move or switch it on the target, ...) is up to the project. The live draft
    stage holds one change: while it waits for acceptance, the next one waits behind it or is
    accepted together with it.

    without draft - software that is not drafted has no draft branch, the flow is release ->
    master and master is the release, as in 1.2.0. The build shifts back to master: Release
    and Release reports run on branch 'master' instead of 'draft', DRAFT_TO_LIVE is not used,
    and master publishes the release and deploys it to the live publish stage. Steps 8-10
    become one step: merge release to master. Hotfixes are merged to master directly.

    hotfix* - branched from master, one fix, quick review, then TEST and PRELIVE, where
    QA validates and verifies it. It deliberately does not deploy to DEV: DEV is the
    development integration target and a hotfix integrates nothing. If QA decides the fix
    needs more testing it is merged to develop, and it reaches DEV through the normal
    development flow from there. A hotfix is merged to draft and draft to master to go live,
    and master is what deploys live and tags - a hotfix never goes live directly.

    No pull request builds.

    [6 branches] x [4 environments, LIVE in live draft and live publish stages]. feature*
    deploys nowhere: it is built and tested only.
    */

    agent any

    triggers {
        // pollSCM, not cron: cron fires on the timer whether or not anything was pushed, so it
        // builds the same commit over and over. pollSCM asks the SCM every 5 minutes and only
        // triggers when there really are new commits. H spreads the poll across the interval so
        // that all jobs do not hit the SCM in the same second.
        //
        // In a MULTIBRANCH pipeline this is redundant and costs more than it gives: the folder
        // already discovers commits by branch indexing, and this makes every branch job poll the
        // repository separately on top of that - N branches, N pollers, all against one remote.
        // The multibranch way is to leave this out and drive builds from either a webhook (best:
        // instant, no polling at all) or the folder's own "Scan Multibranch Pipeline Triggers".
        // It is kept here because this Jenkinsfile is also meant to work as a single branch job,
        // where nothing else would trigger it.
        pollSCM('H/5 * * * *')
    }

    options {
        buildDiscarder(
            logRotator(
                numToKeepStr: '20',
                artifactNumToKeepStr: '10'
            )
        )

        // Commits arrive in bursts, and building every intermediate commit is wasted work.
        // Two different mechanisms are needed, because they solve two different halves:
        //
        // quietPeriod - the burst that has not started building yet. After a trigger Jenkins
        // holds the queue item this long before it becomes buildable, and every further commit
        // inside the window folds into the same build.
        //
        // Both of these are PER JOB, and in a multibranch pipeline every branch is its own job.
        // A quiet period on one branch does not hold another branch back - their windows count
        // down at the same time - but it does delay every branch by this much, which is very
        // visible when several branches are pushed at once. Keep it short: it only has to cover
        // how long a push burst takes, not how long a build takes. Set it to 0 while testing the
        // pipeline itself.
        //
        // disableConcurrentBuilds(abortPrevious: true) - the build that is already running.
        // Without it Jenkins starts a second run beside the first whenever an executor is
        // free, so several intermediate commits build at once. With it the runs of THIS branch
        // are serialised, and abortPrevious kills the older run the moment a newer one is ready:
        // the newest change wins and the superseded ones never finish. Other branches are not
        // affected - if branches are waiting for each other, that is the executor count, not
        // this option.
        quietPeriod(15)
        disableConcurrentBuilds(abortPrevious: true)
    }

    environment {
        // 1 EXAMPLE : PATH setup example
        //PATH = "/opt/setmy.info/bin:$PATH"
        // 2 EXAMPLE : Variables usage
        ABC = 'DEF'
        GHI = "$ABC"

        SMI_PROFILES = 'ci'

        MASTER_TO_LIVE = 'DEPLOY'
        DRAFT_TO_LIVE = 'DEPLOY'

        RELEASE_TO_PRELIVE = 'DEPLOY'
        HOTFIX_TO_PRELIVE = 'DEPLOY'

        DEVELOPMENT_TO_TEST = 'DEPLOY'
        RELEASE_TO_TEST = 'DEPLOY'
        HOTFIX_TO_TEST = 'DEPLOY'

        DEVELOPMENT_TO_DEV = 'DEPLOY'
    }

    stages {
        stage('Inspection') {
            parallel {
                stage('Pre-build') {
                    /*
                    Stage to get into build logs pre build existing environment conditions, versions, getting CI build
                    info into log etc.
                    */
                    steps {
                        echo "Jenkins node: ${env.NODE_NAME}"
                        echo "Operating system: ${isUnix() ? 'Unix/Linux' : 'Windows'}"

                        runCommand 'echo "Hello from command shell"'

                        // 3 EXAMPLE : File based check
                        script {
                            if (!fileExists('README.md')) {
                                error('README.md missing')
                            }
                        }

                        echo 'Pre build inspection and precondition check. Put here commands to check, that build tools are installed.'

                        // 4 EXAMPLE : Variables usage vol 2
                        runCommand 'echo "GHI=${GHI}"'
                        echo "Message GHI=${GHI}"

                        // 5 EXAMPLE : Just emulating longer build
                        sleep 5

                        // 6 EXAMPLE : Repeating example
                        retry(count: 7) {
                            runCommand 'echo "Many times, why?"'
                        }

                        // 7 EXAMPLE : Timeout example
                        timeout(time: 10) {
                            runCommand 'echo "What is this time?"'
                        }

                        // 8 EXAMPLE : email sending
                        emailext(
                            subject: "Jenkins job: $JOB_NAME, build: $BUILD_NUMBER",
                            body: "Job: $JOB_NAME, build: $BUILD_NUMBER, url: ${env.BUILD_URL}",
                            recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                        )
                    }
                }
                /*
                Stage to install build required tools. Build dependencies.
                */
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
                /*
                Stage to install language and source, language code dependencies.
                */
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

        /*
        Stage to build code with with executing all needed steps to measure different type of code quality.
        */
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
                /*
                Stage to push or upload build packages/artifacts to file storage systems.
                */
                stage('Release') {
                    when {
                        branch 'draft'
                        // changeset "**/file/to/be/changed"
                    }
                    steps {
                        echo 'Put here software release steps'
                    }
                }
                stage('Snapshot') {
                    when {
                        branch pattern: 'devel.*', comparator: 'REGEXP'
                    }
                    steps {
                        echo 'Put here software snapshot publishing steps'
                    }
                }
                stage('Release reports') {
                    when {
                        branch 'draft'
                    }
                    steps {
                        echo 'Put here reports publishing steps'
                    }
                }
                stage('Snapshot reports') {
                    when {
                        branch pattern: 'devel.*', comparator: 'REGEXP'
                    }
                    steps {
                        echo 'Put here reports publishing steps'
                    }
                }
            }
        }
        stage('Deploy') {
            parallel {
                /*
                Stages to deploy/install artifacts to different environments.
                */
                stage('dev') {
                    when {
                        environment name: 'DEVELOPMENT_TO_DEV', value: 'DEPLOY'
                        branch pattern: 'devel.*', comparator: 'REGEXP'
                    }
                    steps {
                        echo 'Put here software development installations steps'
                    }
                }
                stage('test') {
                    when {
                        anyOf {
                            allOf {
                                environment name: 'DEVELOPMENT_TO_TEST', value: 'DEPLOY'
                                branch pattern: 'devel.*', comparator: 'REGEXP'
                            }
                            allOf {
                                environment name: 'RELEASE_TO_TEST', value: 'DEPLOY'
                                branch pattern: 'release.*', comparator: 'REGEXP'
                            }
                            allOf {
                                environment name: 'HOTFIX_TO_TEST', value: 'DEPLOY'
                                branch pattern: 'hotfix.*', comparator: 'REGEXP'
                            }
                        }
                    }
                    steps {
                        echo 'Put here software test installations steps'
                    }
                }
                stage('prelive') {
                    when {
                        anyOf {
                            allOf {
                                environment name: 'RELEASE_TO_PRELIVE', value: 'DEPLOY'
                                branch pattern: 'release.*', comparator: 'REGEXP'
                            }
                            allOf {
                                environment name: 'HOTFIX_TO_PRELIVE', value: 'DEPLOY'
                                branch pattern: 'hotfix.*', comparator: 'REGEXP'
                            }
                        }
                    }
                    steps {
                        echo 'Put here software prelive installations steps'
                    }
                }
                stage('live: draft') {
                    when {
                        environment name: 'DRAFT_TO_LIVE', value: 'DEPLOY'
                        branch 'draft'
                    }
                    steps {
                        echo 'Put here software live draft stage installations steps'
                    }
                }
                stage('live: publish') {
                    when {
                        environment name: 'MASTER_TO_LIVE', value: 'DEPLOY'
                        branch 'master'
                    }
                    steps {
                        echo 'Put here software live publish stage installations steps: the release published by draft'
                    }
                }
            }
        }
        /*
        Stage to make SCM tag. As all results are succeeded then tag reflects FULL build success.
        */
        stage('Tag') {
            when {
                environment name: 'MASTER_TO_LIVE', value: 'DEPLOY'
                branch 'master'
            }
            steps {
                echo 'Put here tagging. For example: '
                echo 'smi-new-tag 1.2.3'
                echo 'And logic to get tag from source files for example.'
            }
        }
    }

    post {
        always {
            // junit '**/target/*-reports/*.xml'
            // 9 EXAMPLE : just placeholder for actions after any build.
            runCommand 'echo "Always"'
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
