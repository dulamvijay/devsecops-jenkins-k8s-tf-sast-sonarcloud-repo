pipeline {
  agent any
  tools { 
        maven 'Maven_3_9_6'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=radicaltech -Dsonar.organization=radicaltech -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=d36f6376dea5ee35cb3548d898ed5d2539ac8687'
			}
    }

	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }

	stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("rt")
                 }
               }
            }
    }
	
	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://615299732666.dkr.ecr.us-west-2.amazonaws.com', 'ecr:us-west-2:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}

	stage('Kubernetes Deployment of RT Bugg Web Application') {
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
    agent {
        docker { image 'owasp/zap2docker-stable' }
    }
    steps {
        withKubeConfig([credentialsId: 'kubelogin']) {
            sh('zap.sh -cmd -quickurl http://$(kubectl get services/rtbuggy --namespace=devsecops -o json | jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
            archiveArtifacts artifacts: 'zap_report.html'
        }
    }
}

	   
  }
}
