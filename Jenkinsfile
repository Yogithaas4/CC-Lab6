pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                # Create Docker network (ignore if exists)
                docker network create lab-network || true

                # Remove old backend containers
                docker rm -f backend1 backend2 || true

                # Run backend containers with hostname set
                docker run -d \
                    --name backend1 \
                    --network lab-network \
                    backend-app

                docker run -d \
                    --name backend2 \
                    --network lab-network \
                    backend-app

                # Wait for containers to initialize
                sleep 5
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                # Remove old nginx container
                docker rm -f nginx-lb || true

                # Run nginx container on same network
                docker run -d \
                    --name nginx-lb \
                    --network lab-network \
                    -p 80:80 \
                    nginx

                # Wait for nginx startup
                sleep 3

                # Copy updated nginx config
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                # Reload nginx
                docker exec nginx-lb nginx -s reload

                # Wait to ensure routing works
                sleep 3
                '''
            }
        }

    }

    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs.'
        }
    }
}
