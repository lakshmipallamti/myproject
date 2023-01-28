pipeline {
    agent any
    environment {
        PATH="$PATH:/opt/maven3.8/bin"
    }
    stages {
        stage('SCM') {
            steps {
                git 'https://github.com/lakshmipallamti/myproject.git'
            }
        }
        stage('build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('sonar') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: '1.0-maven-web-app', classifier: '', file: 'target/1.0-maven-web-app.war', type: 'war']], credentialsId: 'nexus', groupId: 'in.sravani', nexusUrl: '13.232.89.43:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'myproject-snapshot', version: '1.0-SNAPSHOT'
            }
        }
        stage('dev-deploy') {
            steps {
                sshagent(['tom']) {
                    sh 'scp -o StrictHostKeyChecking=no target/1.0-maven-web-app.war ec2-user@65.0.99.0:/home/ec2-user/apache-tomcat-9.0.71/webapps'
                }
            }
        }
        stage ('slack notify-dev') {
            steps {
                slackSend channel: 'aws-devops', message: 'dev deployment was successful'
            }
        }
        stage('dev approval') {
            steps {
                echo "taking approval from dev deployment for qa deployment"
                timeout(time:7 , unit:'DAYS') {
                    input message: 'do you want to deploy in ?' , submitter:'jenkinsadmin'
                }
            }
        }
        stage('qa deployment') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-qa', path: '', url: 'http://13.233.100.49:9090/')], contextPath: '/sravani', war: '**/*.war'
            }
        }
        stage('slack notify-QA') {
            steps {
                slackSend channel :'aws-devops' , message: 'qa deployment was successful'
            }
        }
    }
}
