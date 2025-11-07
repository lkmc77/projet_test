pipeline {
    agent any

    triggers {
        githubPush()
    }

    parameters {
        string(name: 'IMAGE_NAME', defaultValue: 'webapp', description: 'Nom de l’image Docker')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Tag de l’image Docker')
    }

    stages {
        stage('Clone') {
            steps {
                slackSend color: '#439FE0', message: '🚀 Début du stage *Clone*'
                checkout scm
                sh 'ls -la'
                slackSend color: 'good', message: '✅ Stage *Clone* terminé'
            }
        }

        stage('Build') {
            steps {
                slackSend color: '#439FE0', message: '🔧 Début du stage *Build* Docker'
                script {
                    def IMAGE = "${params.IMAGE_NAME}:${params.IMAGE_TAG}"
                    sh """
                        rm -rf webapp || true
                        mkdir -p webapp
                        cd webapp

                        cat <<EOF > Dockerfile
FROM ubuntu:22.04
LABEL maintainer="Lowe"
RUN apt-get update && \\
    DEBIAN_FRONTEND=noninteractive apt-get install -y nginx git && \\
    rm -rf /var/www/html/* && \\
    git clone https://github.com/lkmc77/projet_test.git /var/www/html/ && \\
    apt-get clean && rm -rf /var/lib/apt/lists/*
EXPOSE 80
ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]
EOF

                        docker build -t ${IMAGE} .
                    """
                }
                slackSend color: 'good', message: "✅ Stage *Build* terminé → `${params.IMAGE_NAME}:${params.IMAGE_TAG}`"
            }
        }

        stage('Deploy') {
            steps {
                slackSend color: '#439FE0', message: '🚢 Début du stage *Deploy*'
                script {
                    def IMAGE = "${params.IMAGE_NAME}:${params.IMAGE_TAG}"
                    sh """
                        docker stop webapp-container || true
                        docker rm -f webapp-container || true
                        docker run -d --name webapp-container -p 8082:80 ${IMAGE}
                        sleep 5
                    """
                }
                slackSend color: 'good', message: '🌍 Stage *Deploy* terminé → [http://localhost:8082](http://localhost:8082)'
            }
        }
    }

    post {
        success {
            slackSend color: 'good', message: '✅ Pipeline CI/CD *SUCCÈS* !'
        }
        failure {
            slackSend color: 'danger', message: '❌ Pipeline *ÉCHEC* — Vérifie les logs Jenkins'
        }
    }
}
