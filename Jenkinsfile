pipeline {
    agent any

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
                sh '''
                # Create network if not exists
                docker network create app-network || true

                # Remove old containers
                docker rm -f backend1 backend2 || true

                # Run backend containers
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app

                # Wait for Docker DNS + app startup
                sleep 5
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                # Remove old nginx container
                docker rm -f nginx-lb || true

                # Run nginx on same network
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                # Wait for nginx container to fully start
                sleep 5

                # Copy updated config
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                # Reload nginx safely
                docker exec nginx-lb nginx -s reload

                # Extra small wait to stabilize
                sleep 2
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
