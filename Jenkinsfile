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
            
            # Execute deployment, parse JSON output, and save URL
            DEPLOY_OUTPUT=\$(node_modules/.bin/netlify deploy --dir=build --no-build --json)
            echo "\$DEPLOY_OUTPUT" | node -e 'console.log(JSON.parse(require("fs").readFileSync(0, "utf-8")).deploy_url)' > staging_url.txt
            
            echo "Staging URL saved: \$(cat staging_url.txt)"
          """
        }
      }

      stage("Staging E2E") {
        agent {
          docker {
            image "mcr.microsoft.com/playwright:v1.39.0-jammy"
            reuseNode true
          }
        }
        steps {
          sh """
            STAGING_URL=\$(cat staging_url.txt)
            echo "Running Playwright E2E tests against: \$STAGING_URL"
            
            PLAYWRIGHT_TEST_BASE_URL=\$STAGING_URL npx playwright test --reporter=html
          """
        }
        post {
          always {
            publishHTML([
              allowMissing: false,
              alwaysLinkToLastBuild: false,
              keepAll: false,
              reportDir: "playwright-report",
              reportFiles: "index.html",
              reportName: "Playwright E2E",
              reportTitles: "",
              useWrapperFileDirectly: true
            ])
          }
        }
      }

    }
  }
