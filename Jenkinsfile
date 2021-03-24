pipeline {

  agent any

  environment {
    PATH = "/opt/has/bin:$PATH"
    ABC = 'DEF'
    GHI = "$ABC"
  }

  stages {
    stage('Parallel stage I') {
      parallel {

        stage('Stage name A') {
          steps {
            sh 'echo "Hello stage A : ${GHI}"'
            echo 'Message'
          }
        }

        stage('Stage name B') {
          steps {
            sh 'echo "Hello stage B"'
            echo 'Message'
          }
        }

      }
    }

    stage('Parallel stage II') {
      parallel {

        stage('Stage name C') {
          steps {
            echo 'Hello stage B'
            sh 'echo "Hello"'
          }
        }

        stage('Stage name D') {
          steps {
            echo 'Hello Stage D'
            sh 'echo "Hello"'
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
