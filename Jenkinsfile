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
        
        stage('Create Network') {
            steps {
                sh '''
                docker network create app-network || true
                '''
            }
        }
        
        stage('Deploy Backend Containers') {
            steps {
                sh '''
                docker rm -f backend1 backend2 || true
                
                docker run -d \
                  --name backend1 \
                  --network app-network \
                  backend-app
                
                docker run -d \
                  --name backend2 \
                  --network app-network \
                  backend-app
                
                echo "Waiting for backend containers..."
                sleep 10
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true
                
                docker run -d \
                  --name nginx-lb \
                  --link backend1 \
                  --link backend2 \
                  -p 80:80 \
                  nginx
                
                sleep 5
                
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                
                sleep 5
                
                docker exec nginx-lb nginx -s reload
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
