
def appname = "hello-newapp"
def repo = "ofekyamin"
def appimage = "${repo}/${appname}"
def apptag = "${env.BUILD_NUMBER}"

podTemplate(containers: [
    containerTemplate(
        name: 'jnlp',
        image: 'jenkins/inbound-agent',
        ttyEnabled: true
    ),

    containerTemplate(
        name: 'docker',
        image: 'docker:dind',
        command: 'cat',
        ttyEnabled: true,
        privileged: true
    ),

    containerTemplate(
        name: 'trivy',
        image: 'aquasec/trivy:latest',
        command: 'cat',
        ttyEnabled: true
    )
]) {

    node(POD_LABEL) {

        stage('checkout') {
            container('jnlp') {
                sh '/usr/bin/git config --global http.sslVerify false'
                checkout scm
            }
        }

        stage('Build and Scan') {

            parallel(

                'Build': {
                    container('docker') {

                        sleep 5

                        // Start Docker daemon
                        sh '''
                            dockerd > /tmp/dockerd.log 2>&1 &

                            echo "Waiting for Docker daemon..."

                            until docker info > /dev/null 2>&1; do
                                sleep 1
                            done

                            echo "Docker daemon is ready!"
                        '''

                        // Build Docker image
                        echo "Building docker image..."

                        sh "docker build -t ${appimage}:${apptag} ."

                        // Tag image as latest
                        sh "docker tag ${appimage}:${apptag} ${appimage}:latest"

                        // Login to Docker Hub
                        withCredentials([usernamePassword(
                            credentialsId: '835ac9fd-01b0-4605-acb8-74d56ca47c4e',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )]) {

                            sh '''
                                echo "$DOCKER_PASS" | docker login \
                                    -u "$DOCKER_USER" \
                                    --password-stdin
                            '''

        stage('Push Docker Image') {                    
                            // Push versioned image
                            sh "docker push ${appimage}:${apptag}"

                            // Push latest image
                            sh "docker push ${appimage}:latest"
                        }
                    }
                },

                'Trivy FS Test': {
                    container('trivy') {

                        echo "Running Trivy filesystem scan..."

                        sh '''
                            trivy fs \
                                --scanners vuln,secret,misconfig \
                                --severity MEDIUM,HIGH,CRITICAL \
                                .
                        '''
                    }
                }
            )
        }

        stage('Trivy Image Test') {
            container('trivy') {

                echo "Running Trivy image scan..."

                sh """
                    trivy image \
                        --scanners vuln,secret,misconfig \
                        --severity MEDIUM,HIGH,CRITICAL \
                        ${appimage}:${apptag}
                """
            }
        }

        stage('Deploy') {
            container('docker') {

                echo "Deploying ${appimage}:${apptag}..."

                sh """
                    docker pull ${appimage}:${apptag}

                    docker stop ${appname} || true
                    docker rm ${appname} || true

                    docker run -d \
                        --name ${appname} \
                        -p 5000:5000 \
                        ${appimage}:${apptag}
                """

                echo "Deployment completed!"
            }
        }
    }
}