pipeline {
  agent any 
  tools {
    maven 'Maven'
  }

environment {
        GITHUB_TOKEN = credentials('github_token')
        API_KEY = credentials('dojo_api_token')
        DOJO_IP = "35.154.229.151"
    }

stages {
        stage('Cleanup workspace') {
            steps {
                cleanWs()
            }
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
		  sh '''
    			curl -X POST "http://${DOJO_IP}:8080/api/v2/import-scan/" \
                        -H "Authorization: Token ${API_KEY}" \
                        -F "file=@trufflehog_output.json" \
                        -F "scan_type=Trufflehog Scan" \
                        -F "engagement=2" \
                        -F "version=1.0"
                    '''
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
		  
	      sh '''
                    curl -X POST "http://${DOJO_IP}:8080/api/v2/import-scan/" \
                    -H "Authorization: Token ${API_KEY}" \
                    -F "file=@dependency-check-report.xml" \
                    -F "scan_type=Dependency Check Scan" \
                    -F "engagement=1" \
                    -F "version=1.0"
                '''
              
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
                    script {
                        def warFile = '/var/lib/jenkins/workspace/devsecops-pipeline/webgoat-server/target/webgoat-2023.8.jar'
                        
                        sshagent(['app-server']) {
                            sh """
                                scp -o StrictHostKeyChecking=no ${warFile} ubuntu@3.110.210.81:/WebGoat
                            """
                            
                            sh """
                                ssh -o StrictHostKeyChecking=no ubuntu@3.110.210.81 \
                                "nohup java -jar /WebGoat/webgoat-2023.8.jar --server.address=0.0.0.0 > logfile.log 2>&1 & disown"
                            """
                        }
                    }
            }
        }

	

	stage ('DAST - OWASP ZAP') {
            steps {
           sshagent(['deploy-ssh']) {
                    sh '''
                        sleep 30
                        
                        ssh -o StrictHostKeyChecking=no ubuntu@3.111.169.114 \
                        'echo "ubuntu" | sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t zaproxy/zap-stable zap-full-scan.py -t http://3.110.210.81:8080/WebGoat -x zap_report.yml' || true 
                    
                        ssh -o StrictHostKeyChecking=no ubuntu@3.111.169.114 \
                        "curl -X POST 'http://${DOJO_IP}:8080/api/v2/import-scan/' \
                        -H 'Authorization: Token ${API_KEY}' \
                        -F 'scan_type=ZAP Scan' \
                        -F 'file=@/home/ubuntu/zap_report.yml' \
                        -F 'engagement=3' \
                        -F 'version=1.0'"
                    '''
                } 
           }       

    }
  }
}
}
