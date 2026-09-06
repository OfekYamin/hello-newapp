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
    )
]) {
    node(POD_LABEL) {

        stage('checkout') {
            container('jnlp') {
                sh '/usr/bin/git config --global http.sslVerify false'
                checkout scm
            }
        }

        stage('build') {
            container('docker') {

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
                    credentialsId: 835ac9fd-01b0-4605-acb8-74d56ca47c4e
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin
                    '''

                    // Push versioned image
                    sh "docker push ${appimage}:${apptag}"

                    // Push latest image
                    sh "docker push ${appimage}:latest"
                }
            }
        }
    }
}