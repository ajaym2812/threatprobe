pipeline {
  agent any 
  tools {
    maven 'Maven'
  }
  
  stages {
      stage ('Initialize') {
          steps {
          sh '''
          echo "PATH = ${PATH}"
          echo "M2_HOME = ${M2_HOME}"
          ''' 
              
            }
          
        }
      
      stage ('Check secrets') {
          steps {
              sh 'trufflehog3 https://github.com/ajaym2812/threatprobe.git -f json -o truffelhog_output.json || true'
            //  sh './trufflehog_report.sh'
            }
          
        }
      
      // stage ('Software composition analysis') {
      //     steps {
      //     dependencyCheck additionalArguments: ''' 
      //     -o "./" 
      //     -s "./"
      //     -f "ALL" 
      //     --prettyPrint''', odcInstallation: 'OWASP-DC'
      //     dependencyCheckPublisher pattern: 'dependency-check-report.xml'
              
      //     }
          
      // }

      stage ('Static analysis') {
          steps {
              withSonarQubeEnv('sonarqube') {
                  sh 'mvn sonar:sonar'
				  }
				}
			}
			stage ('Generate build') {
			    steps {
			        sh 'mvn clean install ' // -DskipTests'
			        
			    }
			}
		}
	}
