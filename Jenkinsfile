pipeline {
    agent any

    parameters {
        choice(
            name: 'Backend_Count',
            choices: ['1', '2'],
            description: 'Number of backend containers'
        )
    }

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                script {
                    sh "docker network create app-network || true"
                    sh "docker rm -f backend1 backend2 || true"

                    if (params.Backend_Count == '1') {
                        sh "docker run -d --name backend1 --network app-network backend-app"
                    } else {
                        sh "docker run -d --name backend1 --network app-network backend-app"
                        sh "docker run -d --name backend2 --network app-network backend-app"
                    }
                }
            }
        }

        stage('Deploy NGINX') {
            steps {
                script {

                    sh "docker rm -f nginx-lb || true"

                    if (params.Backend_Count == '1') {

                        sh '''
                        echo "upstream backend_servers {
                            server backend1:8080;
                        }

                        server {
                            listen 80;
                            location / {
                                proxy_pass http://backend_servers;
                            }
                        }" > default.conf
                        '''

                    } else {

                        sh '''
                        echo "upstream backend_servers {
                            server backend1:8080;
                            server backend2:8080;
                        }

                        server {
                            listen 80;
                            location / {
                                proxy_pass http://backend_servers;
                            }
                        }" > default.conf
                        '''
                    }

                    sh '''
                    docker run -d \
                      --name nginx-lb \
                      --network app-network \
                      -p 80:80 \
                      nginx

                    docker cp default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                    docker exec nginx-lb nginx -s reload
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
