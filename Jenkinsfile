pipeline {
  agent any 
  tools {
    maven 'Maven'
  }

  stages {
    stage('Initialize') {
      steps {
        script {
          echo "PATH = ${env.PATH}"
          echo "M2_HOME = ${env.M2_HOME}"
        }
      }
    }

    stage('Check Secrets') {
      steps {
        script {
          // Run trufflehog and handle errors appropriately
          sh 'trufflehog3 https://github.com/ajaym2812/threatprobe.git -f json -o truffelhog_output.json'
        }
      }
    }

    stage('Software Composition Analysis') {
      steps {
        dependencyCheck additionalArguments: ''' 
          -o "./" 
          -s "./"
          -f "ALL" 
          --prettyPrint''', 
          odcInstallation: 'OWASP-DC'
        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
      }
    }

    stage('SAST - SonarQube') {
      steps {
        withSonarQubeEnv('sonarqube') {
          // Run SonarQube analysis
          sh 'mvn clean sonar:sonar -Dsonar.java.binaries=src'
        }
      }
    }

    stage('Deploy to Server') {
      steps {
        timeout(time: 3, unit: 'MINUTES') {
          sshagent(['jenkins-ssh-id']) {
            // Deploy and run the application
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.110.210.81 "nohup java -jar /WebGoat/webgoat-2023.8.jar &"'
          }
        }
      }
    }

    stage('DAST - OWASP ZAP') {
      steps {
        sshagent(['dast-server']) {
          // Run OWASP ZAP scan and handle errors
          sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.111.169.114 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t zaproxy/zap-stable zap-full-scan.py -t http://3.110.210.81:8080/WebGoat -x zap_report"'
          // Optionally, run a script to generate the report
          sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.111.169.114 "./zap_report.sh"'
        }
      }
    }
  }

  post {
    always {
      echo 'Pipeline completed.'
    }
    success {
      echo 'Pipeline succeeded!'
    }
    failure {
      echo 'Pipeline failed!'
    }
    unstable {
      echo 'Pipeline is unstable!'
    }
    changed {
      echo 'Pipeline status changed!'
    }
  }
}
