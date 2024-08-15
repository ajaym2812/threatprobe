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
        }
      }
    }

    stage ('Deploy to server') {
      steps {
        timeout(time: 3, unit: 'MINUTES') {
          sshagent(['jenkins-ssh-id']) {
             // sh 'scp -o StrictHostKeyChecking=no target/webgoat-2023.8.jar ubuntu@3.110.210.81:/WebGoat/webgoat-2023.8.jar'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.110.210.81 "nohup java -jar /WebGoat/webgoat-2023.8.jar &"'
          }
        }
      }     
    }

    stage ('DAST - OWASP ZAP') {
      steps {
        sshagent(['dast-server']) {
          sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.111.169.114 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t zaproxy/zap-stable zap-full-scan.py -t http://3.110.210.81:8080/WebGoat -x /zap/wrk/zap_report.xml || true"'
        }
      }       
    }
  }
}
