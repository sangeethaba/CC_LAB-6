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
