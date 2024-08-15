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
	 // stage ('Generate build') {
	 // 	steps {
	 //  		//sh 'sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-11-openjdk-amd64/bin/java 2000'
	 //  		//sh 'sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-17-openjdk-amd64/bin/java 1000'
	 // 		sh 'mvn clean install -DskipTests'
	 // 			
			        
	 		 //  }
	 		//}
	    stage ('Deploy to server') {
              steps {
	  // sh 'mvn clean install -DskipTests'
	     timeout(time: 3, unit: 'MINUTES') {
                sshagent(['jenkins-ssh-id']) {
               // sh 'scp -o StrictHostKeyChecking=no /tmp/webgoat-2023.8.jar ubuntu@ 3.110.210.81:/WebGoat'
		sh 'ssh -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no ubuntu@3.110.210.81 "nohup java -jar /WebGoat/webgoat-server-v8.2.0-SNAPSHOT.jar &"
                    }
	       }
          }     
      }
	}
	}
