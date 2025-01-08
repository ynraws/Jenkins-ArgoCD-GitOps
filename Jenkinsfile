pipeline {
	agent any
	tools {
		nodejs 'NodeJS'
	}
	environment {
		DOCKER_HUB_CREDENTIALS_ID = 'jen-dockerhub'
		DOCKER_HUB_REPO = 'iquantc/iquant-app'
	}
	stages {
		stage('Checkout Github'){
			steps {
				git branch: 'main', credentialsId: 'jenkins-gitops-argocd', url: 'https://github.com/iQuantC/Jenkins-ArgoCD-GitOps.git'
			}
		}		
		stage('Install node dependencies'){
			steps {
				sh 'npm install'
			}
		}
		stage('Test Code'){
			steps {
				sh 'npm test'
			}
		}
		stage('Build Docker Image'){
			steps {
				script {
					dockerImage = docker.build("${DOCKER_HUB_REPO}:latest")
				}
			}
		}
		stage('Trivy Scan'){
			steps {
				//sh 'trivy --severity HIGH,CRITICAL --no-progress image --format table -o trivy-scan-report.txt ${DOCKER_HUB_REPO}:latest'
				sh 'trivy --severity HIGH,CRITICAL --no-progress image --skip-update --format table -o trivy-scan-report.txt ${DOCKER_HUB_REPO}:latest'
			}
		}
		stage('Push Image to DockerHub'){
			steps {
				script {
					docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_HUB_CREDENTIALS_ID}"){
						dockerImage.push('latest')
					}
				}
			}
		}
		stage('Install Kubectl & ArgoCD CLI'){
			steps {
				//sh '''
				//curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                		//chmod +x kubectl
                		//mv kubectl /usr/local/bin/kubectl
				sh '''
				curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
				chmod +x /usr/local/bin/argocd
				'''
			}
		}
		stage('Apply Manifest & Sync App with ArgoCD'){
			steps {
				script {
					kubeconfig(credentialsId: 'kubeconf', serverUrl: 'https://192.168.49.2:8443') {		
    						//sh 'kubectl apply -f manifests/deployment.yaml'
						//sh 'kubectl apply -f manifests/service.yaml'
						sh '''
						argocd login 54.165.246.202:31506 --username admin --password $(kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d) --insecure
						argocd app sync argojenkins
						'''
					}
				}
			}
		}
		//stage('Update GitOps Repo'){
		//	steps {
		//		withCredentials([gitUsernamePassword(credentialsId: 'git-cred', gitToolName: 'Default')]) {

		//			sh '''
		//			sed -i "s/dockerImage/$BUILD_NUMBER/g" manifests/deployment.yaml
		//			git config credential.helper store
		//			echo "https://$GIT_USERNAME:$GIT_PASSWORD@github.com" > ~/.git-credentials
		//			git add .
		//			git commit -m "Updated image to $BUILD_NUMBER"
		//			git push origin main
		//			kubectl apply -f argocd-app.yaml --validate=false
		//			'''
		//		}
		//	}
		//}
		//stage('Deploy via ArgoCD'){
                //        steps {
                //             	sh "kubectl apply -f argocd-app.yaml --validate=false"
                //        }
               //}
	}

	post {
		success {
			echo 'Build&Deploy completed succesfully!'
		}
		failure {
			echo 'Build&Deploy failed. Check logs.'
		}
	}
}
