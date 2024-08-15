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
	 		// }
stage('Deploy to server') {
    steps {
        timeout(time: 3, unit: 'MINUTES') {
            sshagent(['app-server']) {
                sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@3.110.210.81 "
                # Kill the process using port 9090 if any
                if command -v fuser >/dev/null 2>&1; then
                    fuser -k 9090/tcp || true
                else
                    echo 'fuser not found, using lsof instead.'
                    lsof -ti:9090 | xargs kill -9 || true
                fi
                
                # Start the Java application in the background
                nohup java -jar /WebGoat/webgoat-2023.8.jar > /dev/null 2>&1 &
                "
                '''
            }
        }
    }
}

	

	stage ('DAST - OWASP ZAP') {
            steps {
           sshagent(['jenkins-ssh-id']) {
                sh 'ssh -o  StrictHostKeyChecking=no ubuntu@3.111.169.114 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t zaproxy/zap-stable zap-full-scan.py -t http://3.110.210.81:8080/WebGoat -x zap_report || true" '
		
		   //sh 'ssh -o  StrictHostKeyChecking=no apps@10.97.109.243 "sudo docker run --rm -v /home/apps:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://10.97.109.244:8081/WebGoat -x zap_report -n defaultcontext.context || true"'
		sh 'ssh -o  StrictHostKeyChecking=no ubuntu@3.111.169.114 "./zap_report.sh"'
              }      
           }       
    }
  }
}
