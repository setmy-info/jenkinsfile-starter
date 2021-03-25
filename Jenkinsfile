pipeline {

    agent any

    environment {
        PATH = "/opt/has/bin:$PATH"
        ABC = 'DEF'
        GHI = "$ABC"
    }

    stages {
        stage('Inspection') {
            parallel {
                stage('Examples') {
                    steps {
                        echo 'That section can be deleted for real life situations'
                        sh 'echo "GHI=${GHI}"'
                        echo 'Message "GHI=$GHI"'
                        sleep 5
                        retry(count: 7) {
                            sh 'echo "Many times, why?"'
                        }
                        timeout(time: 10) {
                            sh 'echo "What is this time?"'
                            // sh 'exit 1' // Failing that step
                        }
                        // build(job: 'has-web-app-new', propagate: true)
                        emailext (
                            subject: "Jenkins job: $JOB_NAME, build: $BUILD_NUMBER",
                            body: "Job: $JOB_NAME, build: $BUILD_NUMBER, url: ${env.BUILD_URL}",
                            recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                        )
                        fileExists 'README.md'
                    }
                }
                stage('Build tools') {
                    steps {
                        echo 'Put here commands to check, that build tools are installed'
                        sh 'echo "Hello stage B"'
                    }
                }                
            }
        }

        stage('Preparation') {
            parallel {
                stage('Install') {
                    steps {
                        echo 'Installation commands go here'
                        echo 'npm install'
                        echo 'Put here build configuration commands'
                        echo './config'
                    }
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Put here resource copy commands'
                echo 'mvn resources'
                echo 'Put here compilation commands'
                echo 'mvn compile'
                echo 'Put here unit tests'
                echo 'mvn test'
                echo 'Put here unit tests coverage'
                echo 'mvn test'
                echo 'Put here mutation tests coverage'
                echo 'mvn test'
                echo 'Put here packaging'
                echo 'mvn package'
                echo 'Put here local publishing'
                echo 'mvn install'
            }
        }
        
        stage('Validation') {
            parallel {
                stage('Code quality checks') {
                    steps {
                        echo 'Put here findbug/stopbug, style check'
                    }
                }
                stage('Security checks') {
                    steps {
                        echo 'dependencies vulnreability checks'
                    }
                }
                stage('Integration tests') {
                    steps {
                        echo 'Put here integration tests'
                        echo 'mvn verify'
                    }
                }
                stage('System tests') {
                    steps {
                        echo 'Put here system tests'
                        echo 'mvn test'
                    }
                }
                stage('Acceptance tests') {
                    steps {
                        echo 'Put here acceptance tests'
                    }
                }
            }
        }

        stage('Reporting') {
            parallel {
                stage('Site') {
                    steps {
                        echo 'Put here reporting builds steps'
                    }
                }
            }
        }

        stage('Deploy') {
            parallel {
                stage('Software publish') {
                    steps {
                        echo 'Put here software publishing steps'
                    }
                }
                stage('Reports publish') {
                    steps {
                        echo 'Put here reports publishing steps'
                    }
                }
                stage('Install') {
                    steps {
                        echo 'Put here software installations'
                    }
                }
            }
        }
    }

    post {
        always {
            // junit '**/target/*-reports/*.xml'
            sh 'echo "Allways"'
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
