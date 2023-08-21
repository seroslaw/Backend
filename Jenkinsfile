def imageName="seroslaw/panda-backend"
def dockerRegistry=""
def registryCredentials="Dockerhub"
def dockerTag=""

pipeline {
    agent {
        label 'agent'
    }
    
    environment {
        scanerHome = tool 'SonarQube'
    }
    
    stages {
        stage('Git pull') {
            steps {
               //git url: 'https://github.com/seroslaw/Frontend.git', branch: "tests"
            	checkout scm
		}
        }

        stage('Testy jednostkowe') {
            steps {
                sh 'pip3 install -r requirements.txt'
                sh 'python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'
            }
        }
        
        stage('SonarQube') {
            steps {
                withSonarQubeEnv("SonarQube") {
                    sh "${scanerHome}/bin/sonar-scanner"
                }
                timeout(time:1, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true   
                }
            }
        }
        
        stage('Docker') {
            steps {
                script{
                    dockerTag = "RC-${env.BUILD_ID}"
                    applicationImage=docker.build("$imageName:$dockerTag", ".")
                }
            }
        }
        
        stage('Pushing image to Artifactory') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        }
    }
    post{
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }
}
