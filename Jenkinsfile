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
        //stage("Maven Build"){
          //  steps{
            //    sh "mvn package -DskipTests=true"
            //}
        //}
        stage("SonarQube Code Quality"){
           stage('SonarQube analysis') {
    withSonarQubeEnv(credentialsId: 'f225455e-ea59-40fa-8af7-08176e86507a', installationName: 'My SonarQube Server') { // You can override the credential to be used
      sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
    }
  }
}
        stage ("Quality Gate"){
            steps{
                timeout(time:1, unit: 'HOURS'){
                    waitForQualityGate abortPipeline:true
                }
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