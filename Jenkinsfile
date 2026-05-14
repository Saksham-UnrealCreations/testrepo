pipeline {
    agent any
    
    environment {
        // Dynamic Naming logic
        /*APP_NAME     = "orders-service"
          DOCKER_REG   = "my-docker-repo.com"
        */
        // Use Git SHA for the tag to ensure uniqueness
        GIT_SHA      = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        //IMAGE_NAME   = "${DOCKER_REG}/${APP_NAME}:${GIT_SHA}"
        
        // Define Namespaces
        /*SIT_NS  = "sit-preview"
        UAT_NS  = "uat-environment"
        PROD_NS = "production"
        */
        
        // PR Specific Logic
        PR_ID        = "${env.CHANGE_ID}" 
        //RELEASE_NAME = "orders-pr-${PR_ID}"
    }
    
    stages {
        stage('1. Checkout SCM') {
            steps {
                // In Multibranch, 'checkout scm' automatically pulls the 
                // specific branch that triggered the build.
                checkout scm
                echo "Successfully checked out branch: ${env.BRANCH_NAME}"
            }
        }
        
        stage('2. Build & Test') {
            steps {
                echo "Running unit tests for ${env.BRANCH_NAME}..."
                // Example: sh "npm install && npm test"
                sh "echo 'Unit Tests Passed'"
                sh "echo 'Build completed"
            }
        } 
        
        stage('3. SonarQube Quality Gate') {
            // We scan during the PR phase to prevent "bad code" from being merged
            when { changeRequest() }
            steps {
                    //withSonarQubeEnv('MySonarServer') {
                        echo "Analyzing Code Quality..."
                        //sh "${MAVEN_HOME}/bin/mvn sonar:sonar"
            }
        }
        
        stage('4. Docker Build & Push') {
            when { changeRequest() } 
            steps {
                 sh "echo 'Image build and push'"
                //sh "docker build -t ${IMAGE_NAME} ."
                //sh "docker push ${IMAGE_NAME}"
            }
        }
        
        stage('5. Deploy to SIT (Ephemeral)') {
            when { changeRequest() }
            steps {
                // Deploying to a unique namespace per developer PR
                sh "echo 'Depling to SIT server'"
                /*sh """
                helm upgrade --install ${RELEASE_NAME} ./charts/my-app \
                --namespace ${SIT_NS} \
                --set image.tag=${GIT_SHA} \
                --set ingress.host=${RELEASE_NAME}.sit.com
                """
                */
            }
        }
        
        stage('6. Promote to UAT') {
            when { changeRequest() }
            steps {
                timeout(time: 3, unit: 'DAYS') {
                      input message: "SIT Testing complete? No bugs found?", ok: "Proceed to UAT"
                }
                        echo "SIT Passed! Promoting to UAT for Business Sign-off..."
                        //sh "helm upgrade --install ${APP_NAME}-uat ./charts --namespace uat --set image.tag=${UAT_TAG}"
            }
       }
       
       stage('7. UAT Approval') {
            when { changeRequest() }
            steps {
                timeout(time: 5, unit: 'DAYS') {
                    input message: "Does the Business team approve for Production?", ok: "Approve for Merge"
                }
            }
        }
        
        stage('8. Final Release') {
            when { branch 'main' }
            steps {
                script {
                    echo "Final Promotion to Production..."
                    /*sh "docker pull ${DOCKER_REG}/${APP_NAME}:${UAT_TAG}"
                    sh "docker tag ${DOCKER_REG}/${APP_NAME}:${UAT_TAG} ${DOCKER_REG}/${APP_NAME}:${PROD_TAG}"
                    sh "docker push ${DOCKER_REG}/${APP_NAME}:${PROD_TAG}"
                    sh "helm upgrade --install ${APP_NAME}-prod ./charts --namespace production --set image.tag=${PROD_TAG}"
                    */    
                }
            }
        }
    }
    
        post {
        aborted {
            // Triggered if the 5-day timeout hits
            script {
                if (env.CHANGE_ID) {
                    echo "Timeout reached. Deleting SIT Pods for PR-${PR_ID} to save costs."
                    //sh "helm uninstall ${RELEASE_NAME} --namespace ${SIT_NS} || true"
                }
            }
        }
        
        success {
            script {
                if (env.BRANCH_NAME == 'main') {
                    echo "Production Release Successful!"
                }
            }
        }

        always {
            script {
                // If a PR is merged, we clean up the SIT environment automatically
                // This logic would be triggered by a separate cleanup job or GitHub Webhook
                if (isPrMerged()) {
                     echo "PR merged. Deleting SIT Pods for PR-${PR_ID} to save costs."
                    //sh "helm uninstall ${RELEASE_NAME} --namespace ${SIT_NS} || true"
                }
            }
            // Clear local workspace to keep Jenkins server clean
            cleanWs()
        }
    }   
}   
