pipeline {
    agent {
        label "java"
    }
    tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "MAVEN_3.9.6"
    }
	 environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "http://10.168.138.60:8081/"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "VProfile-1"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus-server"
    }
    stages {
        stage('Clone Code') {
            steps {
                git credentialsId: 'saddammohd941_credentials', url: 'https://github.com/saddammohd941/simplecutomerapp.git', branch: 'main'
            }
        }
        stage("mvn build") {
            steps {
                script {
                    // If you are using Windows then you should use "bat" step
                    // Since unit testing is out of the scope we skip them
                    sh 'mvn -Dmaven.test.failure.ignore=true clean install'
                }
            }
        }
        stage('SonarCloud') {
            steps {
                withCredentials([string(credentialsId: 'sonar', variable: 'SONAR_TOKEN')]) {
                    sh """#!/bin/bash
                        export PATH=\$PATH:/opt/sonar-scanner/bin

                        # Clean up any previously bad structure
                        rm -rf src/main/java/src

                        # Create src/main/java if it doesn't exist
                        mkdir -p src/main/java

                        # Copy only .java files NOT already in src/main/java or src/test/java
                        if find src -name "*.java" | grep -q .; then
                        find src -name "*.java" \\
                            ! -path "src/main/java/*" \\
                            ! -path "src/test/java/*" \\
                            -exec cp --parents {} src/main/java/ \\;
                        else
                        echo "No Java files found in src, skipping move."
                        fi

                        # Compile project
                        mvn clean compile

                        echo "Compiled files:"
                        ls -R target/classes

                        # Confirm source directory exists
                        if [ ! -d src/main/java ] || [ -z "\$(ls -A src/main/java)" ]; then
                        echo "src/main/java does not exist or is empty. Skipping SonarCloud analysis."
                        exit 1
                        fi

                        # Run SonarCloud analysis
                        sonar-scanner -X \\
                        -Dsonar.projectKey=simplecutomerapp \\
                        -Dsonar.projectName=simplecutomerapp \\
                        -Dsonar.projectVersion=2.0 \\
                        -Dsonar.organization=saddammohd941 \\
                        -Dsonar.sources=src/main/java \\
                        -Dsonar.binaries=target/classes \\
                        -Dsonar.junit.reportsPath=target/surefire-reports \\
                        -Dsonar.jacoco.reportPaths=target/jacoco.exec \\
                        -Dsonar.login=\$SONAR_TOKEN \\
                        -Dsonar.host.url=https://sonarcloud.io
                    """
                }
            }
        }
        stage("publish to nexus") {
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
    }
}
