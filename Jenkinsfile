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
        stage('Publish') {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;

                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],

                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );

                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
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