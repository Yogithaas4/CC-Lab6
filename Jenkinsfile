
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

        stage('Deploy Backends') {
            steps {
                sh '''
                docker network create lab-network || true

                docker rm -f backend1 backend2 || true

                docker run -d --name backend1 --network lab-network backend-app
                docker run -d --name backend2 --network lab-network backend-app

                sleep 3
                '''
            }
        }

        stage('Start Nginx Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true

                docker run -d --name nginx-lb \
                --network lab-network \
                -p 80:80 nginx

                sleep 2

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }
}
