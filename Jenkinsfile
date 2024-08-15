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
                        // Kill any process using port 9090
                        sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.110.210.81 "sudo fuser -k 9090/tcp || true"'

                        // Start WebGoat on port 9090
                        sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.110.210.81 "nohup java -jar /WebGoat/webgoat-2023.8.jar --server.port=9090 &"'
                    }
                }
            }
        }

        stage ('DAST - OWASP ZAP') {
            steps {
                sshagent(['dast-server']) {
                    // Ensure the ZAP scan targets WebGoat running on port 9090
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.111.169.114 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t zaproxy/zap-stable zap-full-scan.py -t http://3.110.210.81:9090/WebGoat -x zap_report || true"'
                }
            }
        }
    }
}
