pipeline {
    agent any
    
    environment {
        IMAGE_NAME = "hbd"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "adeola-hbd"
        ENV_NAME = "dev"
        VERSION_NAME = "v-0.0.${BUILD_NUMBER}"
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('SonarQube scan') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=DEBOLA-KENDRICK -Dsonar.projectName=DEBOLA-KENDRICK"
                }
            }
        }
        stage("Docker Build") {
            steps {
                sh "docker build -t ${IMAGE_NAME}:v-0.0.${IMAGE_TAG} ."
            }
        }
        stage("Trivy Scan") {
            steps {
                script {
            
                    sh "trivy image ${IMAGE_NAME}:v-0.0.${IMAGE_TAG}"
                }
            }
        }
        // stage("Stop Old Container") {
        //     steps {
        //         sh """
        //             if [ \$(docker ps -q -f name=${CONTAINER_NAME}) ]; then
        //                 echo "Stopping and removing existing container: ${CONTAINER_NAME}"
        //                 docker stop ${CONTAINER_NAME}
        //                 docker rm ${CONTAINER_NAME}
        //             else
        //                 echo "No existing container named ${CONTAINER_NAME} found."
        //             fi
        //         """
        //     }
        // }
        stage("Run App") {
           steps {
            //    sh "docker run -d --name ${CONTAINER_NAME} -p 3436:5000 -e USER=adeola ${IMAGE_NAME}:v-0.0.${IMAGE_TAG}"
            script {
                    kubeconfig(credentialsId: '0035eefa-1fb8-4a7c-83b3-68f7cfa9eff8', serverUrl: '') {
                        // some block
                        sh "sed -i 's|IMAGE_NAME|${IMAGE_NAME}:v-0.0.${IMAGE_TAG}|g' k8s/deploy.yaml"
                        // sh "sed -i 's|ENV_NAME|${ENV_NAME}|g' k8s/deploy.yaml"
                        // sh "sed -i 's|VERSION_NAME|${VERSION_NAME}|g' k8s/deploy.yaml"

                        sh "kubectl apply -f k8s/"
                    }
                }
            }
        }
    }
    post {
        always {
            echo "cleaning up unused image or exited container"
            sh "docker system prune -f"
        }
    }
}