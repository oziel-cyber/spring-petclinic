pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
    }
    environment{
        NEXUS_VERSION = "nexus3"                        
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "localhost:8081"
        NEXUS_REPOSITORY = "maven-nexus-repo"
        NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
        
    }
    stages{
        //stage("Clone code from VCS"){
          //steps{
            //script{
             //git'https://github.com/oziel-cyber/spring-petclinic.git';
        //}
     //}
     //}
        stage("Maven Build"){
            steps{
                sh "mvn package -DskipTests=true"
            }
        }
       stage('sonar-scanner analysis') {
      def sonarqubeScannerHome = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
      withCredentials([string(credentialsId: 'sonar', variable: 'sonarLogin')]) {
        sh "${sonarqubeScannerHome}/bin/sonar-scanner -X -Dsonar.host.url=http://sonarqube:9000 -Dsonar.login=e06fe7b117b183f1360431c59148553bb6a16b0b -Dsonar.projectName=My Sonarqube -Dsonar.projectVersion=${env.BUILD_NUMBER} -Dsonar.projectKey=2f342910c7a2a976d734db0e9d122cef58ea10237b795c759095311c4156b17b  -Dsonar.sources=src/main/java -Dsonar.java.libraries=target/* -Dsonar.java.binaries=target/classes -Dsonar.language=java"
      }
    sh "sleep 40"
    env.WORKSPACE = pwd()
    def file = readFile "${env.WORKSPACE}/.scannerwork/report-task.txt"
    echo file.split("\n")[5]
    
    def resp = httpRequest file.split("\n")[5].split("l=")[1]
    
    ceTask = readJSON text: resp.content
    echo ceTask.toString()
    
    def response2 = httpRequest url : 'http://localhost:9000' + "/api/qualitygates/project_status?analysisId=" + ceTask["task"]["analysisId"]
    def qualitygate =  readJSON text: response2.content
    echo qualitygate.toString()
    if ("ERROR".equals(qualitygate["projectStatus"]["status"])) {
        echo "Build Failed"
        }
       else {
        echo "Build Passed"
    //   build_job("Trigger the delivery job")		
         }
        }
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    //pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "./target/spring-petclinic-2.4.2.jar");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}";
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: org.springframework.samples , packaging: jar , version 2.4.2";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL, 
                            nexusUrl: NEXUS_URL,
                            groupId: org.springframework.samples,
                            version: "2.4.2",
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: spring-petclinic,
                                classifier: '',
                                file: artifactPath,
                                type: jar],
                                [artifactId: spring-petclinic,
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
    }
}