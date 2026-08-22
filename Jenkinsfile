pipeline {
  agent any

  environment {
    NETLIFY_SITE_ID = "a2645abf-c3e5-4982-9a53-7b602ee38d59"
    NETLIFY_AUTH_TOKEN = credentials("netlify-token")
    REACT_APP_VERSION = "1.0.$BUILD_ID"
  }

  stages {

    stage("Build Docker Image") {
      steps {
        sh "docker build -t my-playwright ."
      }
    }

    // stage("Build Microservice") {
    //   agent {
    //     docker {
    //       image "node:18-alpine"
    //       reuseNode true
    //     }
    //   }
    //   steps {
    //     sh """
    //       ls -la
    //       node --version
    //       npm --version
    //       npm ci
    //       npm run build
    //       ls -la
    //     """
    //   }
    // }

    stage("Tests") {
      parallel {
        stage("Unit Tests") {
          agent {
            docker {
              image "node:18-alpine"
              reuseNode true
            }
          }

          steps {
            sh """
              npm test
            """
          }
          post {
            always {
              junit "test-results/junit.xml"
            }
          }
        }
      }
    }

    stage("Deploy Staging") {
      agent {
        docker {
          image "my-playwright"
          reuseNode true
        }
      }

      environment {
        CI_ENVIRONMENT_URL = "https://admirable-jalebi-b289f9.netlify.app"
      }

      steps {
        sh """
          netlify --version
          echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
          netlify status
          netlify deploy --dir=build --json > deploy-output.json
          CI_ENVIRONMENT_URL=\$(node-jq -r ".deploy_url" deploy-output.json)
          echo "${CI_ENVIRONMENT_URL}"
          # npx playwright test --reporter=html
        """
      }

      // post {
      //   always {
      //     publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: "playwright-report", reportFiles: "index.html", reportName: "Staging E2E", reportTitles: "", useWrapperFileDirectly: true])
      //   }
      // }
    }

    // stage('Staging E2E') {
    //   agent {
    //     docker {
    //       image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
    //       reuseNode true
    //     }
    //   }

    //   environment {
    //     CI_ENVIRONMENT_URL = "https://admirable-jalebi-b289f9.netlify.app"
    //   }

    //   steps {
    //     sh '''
    //       npx playwright test --reporter=html
    //     '''
    //   }

    //   post {
    //     always {
    //       publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
    //     }
    //   }
    // }

    // stage("Approval") {
    //   options {
    //     timeout(time: 15, unit: "MINUTES")
    //   }
    //   steps {
    //     script {
    //       input message: "Do you wish to deploy to production?",
    //             ok: "Yes, I am sure!"
    //     }
    //   }
    // }

    // stage("Deploy Prod") {
    //   agent {
    //     docker {
    //       image "my-playwright"
    //       reuseNode true
    //     }
    //   }

    //   environment {
    //     CI_ENVIRONMENT_URL = "https://admirable-jalebi-b289f9.netlify.app"
    //   }

    //   steps {
    //     sh """
    //       node --version
    //       netlify --version
    //       echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
    //       netlify status
    //       netlify deploy --dir=build --prod
    //       npx playwright test  --reporter=html
    //     """
    //   }

    //   post {
    //     always {
    //       publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: "playwright-report", reportFiles: "index.html", reportName: "Prod E2E", reportTitles: "", useWrapperFileDirectly: true])
    //     }
    //   }
    // }

    // stage("Prod E2E") {
    //   agent {
    //     docker {
    //       image "mcr.microsoft.com/playwright:v1.39.0-jammy"
    //       reuseNode true
    //     }
    //   }

    //   environment {
    //     CI_ENVIRONMENT_URL = "https://admirable-jalebi-b289f9.netlify.app"
    //   }

    //   steps {
    //     sh """
    //       npx playwright test  --reporter=html
    //     """
    //   }

    //   post {
    //     always {
    //       publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: "playwright-report", reportFiles: "index.html", reportName: "Playwright E2E", reportTitles: "", useWrapperFileDirectly: true])
    //     }
    //   }
    // }
  }
}
