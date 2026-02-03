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
			      pwd
			      ls -al
			      ls -al k8s || true
			
			      echo "=== find deployment yaml ==="
			      DEPLOY_FILE=$(find . -maxdepth 5 -type f \\( -name "deployment.yaml" -o -name "deployment.yml" \\) | head -n 1)
			      echo "DEPLOY_FILE=$DEPLOY_FILE"
			
			      if [ -z "$DEPLOY_FILE" ]; then
			        echo "deployment.yaml(.yml) 못 찾음. 레포에 파일이 없거나 경로가 다름."
			        exit 1
			      fi
			
			      kubectl --kubeconfig=/var/lib/jenkins/.kube/config apply -f "$DEPLOY_FILE"
			
			      kubectl --kubeconfig=${KUBECONFIG_PATH} rollout restart deployment totalapp-deployment
			      kubectl --kubeconfig=${KUBECONFIG_PATH} get pods
			    '''
		    }
		}
    }
}