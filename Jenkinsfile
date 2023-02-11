pipeline {
    agent any
    environment {
        PATH = "/opt/maven/apache-maven-3.8.7/bin:$PATH"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "192.168.0.193:8081"
        NEXUS_REPOSITORY = "jitsNex-release"
        NEXUS_REPO_ID = "jitsNex-release"
        NEXUS_CREDENTIAL_ID = "nexus_cred"
        ARTVERSION = "${env.BUILD_ID}"
    }
    stages {
        stage("hello"){
            steps{
                echo "hello prajeet, welcome to AbuDhabi"
            }
        }
        stage("git clone"){
            steps{
                git branch: 'main', credentialsId: 'cda3ca04-162b-4e6d-a78b-70626d319515', url: 'https://github.com/pjitss/jits-tomcat-23.git'
            }
        }
        stage("maven clean"){
            steps{
                sh "mvn clean"
            }
        }
        stage("maven install"){
            steps{
                sh "mvn clean install"
            }
        }
        stage('Sonar Scan'){
            steps{
                withSonarQubeEnv('SonarServer') {
                sh "mvn sonar:sonar"
                }
            }
        }
        stage("login & deploy"){
            steps{
                sshagent(['JenkinLogin']) {
                    sh 'ssh -o StrictHostKeyChecking=no root@192.168.0.233 uname -a'
                    sh 'whoami'
                    sh 'hostname'
                    sh "pwd"
                    sh "ls -ltra"
                    sh "scp -o StrictHostKeyChecking=no webapp/target/webapp.war root@192.168.0.233:/opt/tomcat/latest/webapps"
                }
                        
            }
        }
    }
}