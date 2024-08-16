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

        stage('Initialize') {
            steps {
                sh '''
                echo "PATH = ${PATH}"
                echo "M2_HOME = ${M2_HOME}"
                '''
            }
        }

        stage('Check secrets') {
            steps {
                sh '''
                trufflehog https://github.com/ajaym2812/threatprobe.git --json > trufflehog_output.json || true
                '''
                // Upload Trufflehog results to Dojo (uncomment if needed)
                // sh '''
                // curl -X POST "http://${DOJO_IP}:8080/api/v2/import-scan/" \
                //     -H "Authorization: Token ${API_KEY}" \
                //     -F "file=@trufflehog_output.json" \
                //     -F "scan_type=Trufflehog Scan" \
                //     -F "engagement=2" \
                //     -F "version=1.0"
                // '''
            }
        }

        stage('Software Composition Analysis') {
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

        stage('SAST - SonarQube') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn clean sonar:sonar -Dsonar.java.binaries=src'
                }
            }
        }

        // Uncomment if needed
        // stage('Generate Build') {
        //     steps {
        //         sh 'mvn clean install -DskipTests'
        //     }
        // }

        stage('Deploy to Server') {
            steps {
                script {
                    def warFile = '/var/lib/jenkins/workspace/devsecops-pipeline/webgoat-server/target/webgoat-2
