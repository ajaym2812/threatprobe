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
      
      stage ('Software composition analysis') {
          steps {
          dependencyCheck additionalArguments: ''' 
          -o "./" 
          -s "./"
          -f "ALL" 
          --prettyPrint''', odcInstallation: 'OWASP-DC'
          dependencyCheckPublisher pattern: 'dependency-check-report.xml'
              
          }
          
      }

   stage ('SAST - SonarQube') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh 'mvn clean sonar:sonar -Dsonar.java.binaries=src'
	  //sh 'sudo python3 sonarqube.py'
	  //sh './sonarqube_report.sh'
        }
      }
    }
			stage ('Generate build') {
			    steps {
			        sh 'mvn clean install -DskipTests'
				//sh 'mvn sonar:sonar -Dsonar.java.binaries=target/classes'

			        
			   }
			}
	 //    stage ('Deploy to server') {
  //             steps {
	 // // sh 'mvn clean install -DskipTests'
	 //     timeout(time: 3, unit: 'MINUTES') {
  //               sshagent(['app-server']) {
  //                sh 'scp -o StrictHostKeyChecking=no /home/ubuntu/web/webgoat-2023.8.jar ubuntu@ 3.110.210.81:/WebGoat'
		//  sh 'ssh -o  StrictHostKeyChecking=no ubuntu@3.110.210.81 "nohup java -jar /WebGoat/webgoat-2023.8.jar &"'
  //                   }
	 //       }
  //         }     
  //     }
	}
	}
