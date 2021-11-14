pipeline{

	agent any

	environment {
        REGISTRY_URL = "https://registry.hub.docker.com/"
        VERSION_TAG = "0.1.${env.BUILD_ID}"
        IMAGE_NAME = "abdessamii/user-py"
	}
    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '30'))
        disableConcurrentBuilds()
    }
	stages {

		stage('Build Image') {

			steps {
			//Build the image
            docker.build("${IMAGE_NAME}:${VERSION_TAG}","--no-cache  -f ./server/Dockerfile ./server")
			}
		}

    stage('Publish test result') {
        // run local unit test 
        sh "touch tests/*.xml"
        junit 'tests/*.xml'
    }

		stage('SECURITY scan images with trivy') {
            steps {
                script {
                        // clearing and refreshing  the trivy DB cache and scan the image for CRITICAL and HIGH vulnerabilities
                        sh """
                            trivy image --clear-cache
                            time trivy --download-db-only --no-progress --cache-dir .trivycache/	 
                            trivy image --exit-code 0 --severity CRITICAL,HIGH ${IMAGE_NAME}:${VERSION_TAG}
                        """
                        
                        def scann = sh(script: 
                        "trivy image --exit-code 1 --severity CRITICAL,HIGH ${IMAGE_NAME}:${VERSION_TAG} ", 
                        returnStatus: true)
            
                        if (scann != 0 ) {
                  
                        echo "############################################  
                              Image scanning failed. Some vulnerabilities found contains CRITICAL or HIGH CVE  
                              ############################################"
	                    sh "exit 1"
				        }
                        else {
                        echo  "############################################  
                               Image is scanned Successfully. No vulnerabilities found  
                               ###########################################"
                        }    
				
	            }
            }
        }

		stage('Push Image to Registry') {
    		steps {
              script {
                //Push the image to the Dockerhub registry
                docker.withRegistry("${REGISTRY_URL}", '7835b570-ae17-41a9-9998-f670aea2e910') {
                  sh "docker push ${env.IMAGE_NAME.toLowerCase()}:${VERSION_TAG}"

                }
              }
	    	}
		}


	}

	post {
      // Email notification for success
      success {
          emailext body: "Do Not Reply , Build  Image OK Project: ${env.JOB_NAME} Build Number: ${env.BUILD_NUMBER}    URL de build: ${env.BUILD_URL}", recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], subject: 'Success: Job Jenkins IMAGE Build '
      }
      // Email notification for Failure
      failure {
          emailext body: "Do Not Reply , Build  Image KO Project: ${env.JOB_NAME} Build Number: ${env.BUILD_NUMBER}    URL de build: ${env.BUILD_URL}", recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], subject: 'Failure: Job Jenkins IMAGE Build '
      }
      // Cleanup Workspace Jenkins
		always {
			cleanWs()
		}
	}

}

