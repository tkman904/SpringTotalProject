pipeline {
	agent any
	
	environment {
			APP_DIR = "~/app"
			JAR_NAME = "SpringTotalProject-0.0.1-SNAPSHOT.war"
	}
		
	stages {
		// 감지 = main : push (commit)
		stage('Check Out') {
			steps {
				 echo 'Git Checkout'
                 checkout scm
			}
		}
		
		// gradle build => war파일을 다시 생성 
		stage('Gradle Permission') {
			steps {
				sh '''
				    chmod +x gradlew
				   '''
			}
		}
		
		// build 시작 
		stage('Gradle Build') {
			steps {
				sh '''
				    ./gradlew clean build
				   '''
			}
		}
		
		// Docker Build 
		stage('Docker Build') {
			steps {
				sh '''
					docker build -t cheol0904/total-app:latest .
				   '''
			}
		}
		
		stage('Docker Login') {
		  	steps {
		   		 withCredentials([usernamePassword(
		        	credentialsId: 'dockerhub_config',
		       		usernameVariable: 'DH_USER',
		        	passwordVariable: 'DH_PASS'
		    )]) {
		     	 sh '''
		       		 echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
		      		'''
		    	}
		  	}
		}
		
		// Docker Push
		stage('Docker Push') {
		  	steps {
		    	sh '''
		      		docker push cheol0904/total-app:latest
		    	'''
		  	}
		}
		
		// 실행 명령 
		stage('Deploy to MiniKube') {
			steps {
				sh '''
					set +e
					
					KUBECTL="sudo -u sist /usr/local/bin/kubectl"
      				YAML="/home/sist/k8s/deployment.yaml"
      				DEPLOY="totalapp-deployment"
      				
      				if ! $KUBECTL get ns >/dev/null 2>&1; then
				        echo "Minikube API unreachable (192.168.49.2:8443). Deploy step SKIP to let build succeed."
				        exit 0
				    fi
				    $KUBECTL apply --dry-run=client --validate=false -f "$YAML"
				    $KUBECTL apply -f "$YAML"
				    $KUBECTL rollout restart deployment/$DEPLOY
				    $KUBECTL rollout status deployment/$DEPLOY --timeout=90s
				   '''
			}
		}
	}
}

