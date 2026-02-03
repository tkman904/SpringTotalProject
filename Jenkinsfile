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
			      kubectl --kubeconfig=/var/lib/jenkins/.kube/config apply -f /home/sist/k8s/deployment.yaml
			      kubectl --kubeconfig=/var/lib/jenkins/.kube/config rollout restart deployment totalapp-deployment
				  kubectl --kubeconfig=/var/lib/jenkins/.kube/config get pods
			    	'''
		    }
		}
    }
}