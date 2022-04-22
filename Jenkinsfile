def scmVars

pipeline {

  agent {
    kubernetes{
        yaml """
apiVersion: v1
kind: Pod
spec:
	containers:
	- name: docker
		image: docker:20.10.3-dind
		command:
		- dockerd
		- --host=unix:///var/run/docker.sock
		- --host=tcp://0.0.0.0:2375
		- --storage-driver=overlay2
		ttv: true
    securityContext:
			priviledged: true
  - name: helm
    image: lachlanevenson/k8s-helm:v3.5.0
    command:
    - cat
    tty: true
""" 
    } // End kubernetes
  } // End agent

  environment{
    ENV_NAME = "${BRANCH_NAME == "master" ? "uat" : "${BRANCH_NAME}"}"
  }

	stages{

		stage('Clone reviews source code'){
			steps{
				container('jnlp'){
					script{
						scmVars = git branch: '${BRANCH_NAME}',
													credentialsId: 'bookinfo-git-deploy-key',
													url: 'git@github.com:NattaphonMek/sitkmutt-bookinfo-reviews.git'
					}// End Script
				}// End Container
			}// End Step
		}// End Stage

		stage('Build reviews Docker Image and push'){
			steps{
				container('docker'){
					script{
						docker.withRegistry('https://ghcr.io','registry-bookinfo'){
							docker.build("ghcr.io/nattaphonmek/bookinfo-reviews:${ENV_NAME}").push()
						}// End docker.withRegistry
					}// End script
				}// End container
			}// End steps
		}// End stage

    stage('Deploy reviews with helm chart'){
      steps{
        container('helm'){
          script{
            withKubeConfig([credentialId:'gke-kubeconfig']) {
              withCredentials([file(credentialId:'gke-sa-key-json', variable:'GOOGLE_APPLICATION_CREDENTIALS')]){
                sh "helm upgrade -f k8s/helm-values/values-bookinfo-${ENV_NAME}-reviews.yaml --wait \
                --set extraEnv.commit_ID=${scmVars.GIT_COMMIT} \
                --namespace bookinfo-${ENV_NAME} bookinfo-${ENV_NAME}-reviews k8s/helm"
              }// End withCredentials
            }// End withKubeConfig
          }// End script
        }// End container
      }// End steps
    }// End stage

	}// End Stages

} // End Pipeline