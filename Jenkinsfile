  pipeline {
    agent any

    environment {
      NETLIFY_SITE_ID = "a2645abf-c3e5-4982-9a53-7b602ee38d59"
      NETLIFY_AUTH_TOKEN = credentials("netlify-token")
    }

    stages {

      stage("Build") {
        agent {
          docker {
            image "node:18-alpine"
            reuseNode true
          }
        }
        steps {
          sh """
            ls -la
            node --version
            npm --version
            npm ci
            npm run build
            ls -la
          """
        }
      }

      stage("Tests") {
        parallel {
          stage("Unit tests") {
            agent {
              docker {
                image "node:18-alpine"
                reuseNode true
              }
            }

            steps {
              sh """
                #test -f build/index.html
                npm test
              """
            }
            post {
              always {
                junit "test-results/junit.xml"
              }
            }
          }

          stage("E2E") {
            agent {
              docker {
                image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                reuseNode true
              }
            }

            steps {
              sh """
                npm install serve
                node_modules/.bin/serve -s build &
                sleep 10
                npx playwright test  --reporter=html
              """
            }

            post {
              always {
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: "playwright-report", reportFiles: "index.html", reportName: "Playwright HTML Report", reportTitles: "", useWrapperFileDirectly: true])
              }
            }
          }
        }
      }

      stage("Deploy Staging") {
        agent {
          docker {
            image "node:18-alpine"
            reuseNode true
          }
        }
        steps {
          sh """
            npm install netlify-cli
            npm install netlify-cli node-jq
            node_modules/.bin/netlify --version
            echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
            node_modules/.bin/netlify status
            node_modules/.bin/netlify deploy --dir=build --no-build --json > "deploy-output.json"
          """

          script {
            env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
          }
        }
      }

      stage('Staging E2E') {
        agent {
          docker {
            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
            reuseNode true
          }
        }

        environment {
          CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
        }

        steps {
          sh '''
            npx playwright test  --reporter=html
          '''
        }

        post {
          always {
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
          }
        }
      }

      stage("Approval") {
        options {
          timeout(time: 15, unit: "MINUTES")
        }
        steps {
          script {
            input message: "Do you wish to deploy to production?",
                  ok: "Yes, I am sure!"
          }
        }
      }

      stage("Deploy Prod") {
        agent {
          docker {
            image "node:18-alpine"
            reuseNode true
          }
        }
        steps {
          sh """
            npm install netlify-cli
            node_modules/.bin/netlify --version
            echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
            node_modules/.bin/netlify status
            node_modules/.bin/netlify deploy --dir=build --prod --no-build
          """
        }
      }

      stage("Prod E2E") {
        agent {
          docker {
            image "mcr.microsoft.com/playwright:v1.39.0-jammy"
            reuseNode true
          }
        }

        environment {
          CI_ENVIRONMENT_URL = "https://admirable-jalebi-b289f9.netlify.app"
        }

        steps {
          sh """
            npx playwright test  --reporter=html
          """
        }

        post {
          always {
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: "playwright-report", reportFiles: "index.html", reportName: "Playwright E2E", reportTitles: "", useWrapperFileDirectly: true])
          }
        }
      }
    }
  }
