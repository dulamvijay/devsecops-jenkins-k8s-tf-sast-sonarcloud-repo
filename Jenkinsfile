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
  }
}
