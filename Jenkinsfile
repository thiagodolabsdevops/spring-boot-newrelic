pipeline {
    agent {
        kubernetes {
            label 'my-pod-label'
            defaultContainer 'gradle'
            yaml """
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: gradle
                    image: gradle:jdk17
                    command:
                    - 'cat'
                    tty: true
                  - name: docker
                    image: docker
                    command:
                    - 'cat'
                    tty: true
                  - name: kubectl
                    image: lachlanevenson/k8s-kubectl:latest
                    command:
                    - 'cat'
                    tty: true
                  - name: helm
                    image: lachlanevenson/k8s-helm:latest
                    command:
                    - 'cat'
                    tty: true
            """
        }
    }

    stages {
        stage('Download and Run Snyk CLI') {
            steps {
                sh '''
                    latest_version=$(curl -Is "https://github.com/snyk/snyk/releases/latest" | grep "^location" | sed s#.*tag/##g | tr -d "\r")
                    echo "Downloading Snyk CLI Version: ${latest_version}"

                    curl -Lo ./snyk https://github.com/snyk/snyk/releases/download/${latest_version}/snyk-linux
                    chmod +x snyk

                    ./snyk test
                '''
            }
        }

        stage('Build and Test') {
            steps {
                sh '''
                    ./gradlew build
                    ./gradlew test
                '''
            }
            post {
                success {
                    archiveArtifacts 'build/libs/*.jar'
                }
                always {
                    junit 'build/test-results/**/*.xml'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
