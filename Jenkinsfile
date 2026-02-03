pipeline {
    agent any
    
    environment {
        APP_DIR = "~/app"
        JAR_NAME = "SpringTotalProject-0.0.1-SNAPSHOT.war"
        // sist 사용자의 kube 설정을 그대로 참조하도록 환경변수 설정
        KUBECONFIG_PATH = "/home/sist/.kube/config" 
    }
        
    stages {
        stage('Check Out') {
            steps {
                 checkout scm
            }
        }
        
        stage('Gradle Build') {
            steps {
                sh '''
                    chmod +x gradlew
                    ./gradlew clean build -x test
                '''
            }
        }
        
        stage('Docker Build') {
            steps {
                sh 'docker build -t ouxthm/total-app:latest .'
            }
        }
          
        stage('Deploy to Minikube') {
            steps {
                sh '''
                    # 1. sist 계정의 config 권한이 있다면 이 경로를 직접 사용
                    # 2. deployment.yaml 경로를 sist 계정의 절대 경로로 지정
                    kubectl --kubeconfig=/var/lib/jenkins/.kube/config apply -f "$WORKSPACE/k8s/deployment.yaml"
                    
                    # 3. 아까 확인한 실제 Deployment 이름(totalapp-deployment)으로 재시작
                    kubectl --kubeconfig=${KUBECONFIG_PATH} rollout restart deployment totalapp-deployment
                    
                    # 4. 상태 확인
                    kubectl --kubeconfig=${KUBECONFIG_PATH} get pods
                '''
            }
        }
    }
}