pipeline {
  agent any
  tools { 
        maven 'Maven_3_2_5'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=reddy1229
 -Dsonar.organization=reddy1229
 -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=6f2c68300b2d92a6c6316cf401e63ac2d922a2ee'
			}
    }

	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }

	stage('DockerBuild') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("asg")
                 }
               }
            }
    }

	stage('PushDockerImageInToAwsEcr') {
            steps {
                script{
                    docker.withRegistry('https://913524934403.dkr.ecr.us-east-1.amazonaws.com/', 'ecr:us-east-1:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}
	   
	stage('Kubernetes Deployment of ASG Bugg Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
		}
	      }
   	}

	 stage ('wait_for_testing'){
	   steps {
		   sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'
	   	}
	   }
	   
	stage('RunDASTUsingZAP') {
          steps {
		    withKubeConfig([credentialsId: 'kubelogin']) {
				sh('zap.sh -cmd -quickurl http://$(kubectl get services/asgbuggy --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
				archiveArtifacts artifacts: 'zap_report.html'
		    }
	     }
       }   

  }
}
