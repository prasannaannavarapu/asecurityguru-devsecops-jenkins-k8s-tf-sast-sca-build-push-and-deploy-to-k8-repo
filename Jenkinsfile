pipeline {
  agent any
  tools { 
        maven 'Maven_3_2_5'  
    }
   environment {
        SKIP_TESTS = false // Set this to true to skip the test stage
    }	
   stages{
    stage('CompileandRunSonarAnalysis') {
	    when {
                expression { env.SKIP_TESTS == true } // Skip if SKIP_TESTS is true
            }
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=reddybuggywebapp -Dsonar.organization=reddybuggywebapp -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=5ab9cec94adb23137077f6121c5502710d39f06f'
			}
    }

	stage('RunSCAAnalysisUsingSnyk') {
		when {
                expression { env.SKIP_TESTS == true } // Skip if SKIP_TESTS is true
            }
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }

	stage('DockerBuild') { 
		when {
                expression { env.SKIP_TESTS == true } // Skip if SKIP_TESTS is true
            }
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("asg")
                 }
               }
            }
    }

	stage('PushDockerImageInToAwsEcr') {
		when {
                expression { env.SKIP_TESTS == true } // Skip if SKIP_TESTS is true
            }
            steps {
                script{
                    docker.withRegistry('https://913524934403.dkr.ecr.us-east-1.amazonaws.com/', 'ecr:us-east-1:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}
	   
	stage('Kubernetes Deployment of ASG Bugg Web Application') {
		when {
                expression { env.SKIP_TESTS == false } // Skip if SKIP_TESTS is true
            }
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
		}
	      }
   	}

	 stage ('wait_for_testing'){
		 when {
                expression { env.SKIP_TESTS == true } // Skip if SKIP_TESTS is true
            }
	   steps {
		   sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'
	   	}
	   }
	   
	stage('RunDASTUsingZAP') {
		when {
                expression { env.SKIP_TESTS == true } // Skip if SKIP_TESTS is true
            }
          steps {
		    withKubeConfig([credentialsId: 'kubelogin']) {
				sh('zap.sh -cmd -quickurl http://$(kubectl get services/asgbuggy --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
				archiveArtifacts artifacts: 'zap_report.html'
		    }
	     }
       }   

  }
}
