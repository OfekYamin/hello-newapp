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

                echo "Building ${appimage}:${apptag}..."

                sh """
                    docker build -t ${appimage}:${apptag} .
                    docker tag ${appimage}:${apptag} ${appimage}:latest
                """

                echo "Logging into Docker Hub..."

                withCredentials([usernamePassword(
                    credentialsId: '835ac9fd-01b0-4605-acb8-74d56ca47c4e',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login \
                            -u "\$DOCKER_USER" \
                            --password-stdin

                        docker push ${appimage}:${apptag}
                        docker push ${appimage}:latest
                    """
                }
            }
        }
    }
}