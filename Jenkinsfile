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
        NEXUS_REPOSITORY = "maven-releases"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus-server"
    }
    stages {
        stage('Clone Code') {
            steps {
                git credentialsId: 'saddammohd941_credentials', url: 'https://github.com/saddammohd941/simplecustomerapp.git', branch: 'main'
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

                        # Compile, test, and package to generate classes and coverage reports
                        mvn clean package

                        echo "Compiled files:"
                        ls -R target/classes

                        echo "JaCoCo report files:"
                        ls -R target/site/jacoco

                        # Run SonarCloud analysis
                        sonar-scanner -X \\
                        -Dsonar.projectKey=simplecustomerapp \\
                        -Dsonar.projectName=simplecustomerapp \\
                        -Dsonar.projectVersion=2.0 \\
                        -Dsonar.organization=saddammohd941 \\
                        -Dsonar.sources=src/main/java \\
                        -Dsonar.tests=src/test/java \\
                        -Dsonar.java.binaries=target/classes \\
                        -Dsonar.junit.reportsPath=target/surefire-reports \\
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml \\
                        -Dsonar.login=\$SONAR_TOKEN \\
                        -Dsonar.host.url=https://sonarcloud.io
                    """
                }
            }
        }
        stage('Publish to Nexus') {
            steps {
                withEnv(["JAVA_HOME=${tool 'JDK_17'}", "PATH+JAVA=${tool 'JDK_17'}/bin"]) {
                    withMaven(maven: 'MAVEN_3.9.6') {
                        script {
                            def pom = readMavenPom file: 'pom.xml'
                            nexusArtifactUploader(
                                nexusVersion: 'nexus3',
                                protocol: 'http',
                                nexusUrl: '10.168.138.60:8081',
                                groupId: pom.groupId,
                                version: pom.version,
                                repository: pom.version.endsWith('-SNAPSHOT') ? 'maven-snapshots' : 'maven-releases',
                                credentialsId: 'nexus-server',
                                artifacts: [
                                    [artifactId: pom.artifactId, classifier: '', file: "target/${pom.artifactId}-${pom.version}.war", type: 'war']
                                ]
                            )
                        }
                    }
                }
            }
        }
	stage('Deploy to Tomcat') {
            steps {
                sshagent(['tomcat-credentials']) {
                    sh """
                        ls -l target/
                        scp -P 2222 -o StrictHostKeyChecking=no target/SimpleCustomerApp-1.0.0-SNAPSHOT.war root@10.168.133.22:/opt/tomcat/webapps/
                        ssh -p 2222 -o StrictHostKeyChecking=no root@10.168.133.22 "chown tomcat:tomcat /opt/tomcat/webapps/SimpleCustomerApp-1.0.0-SNAPSHOT.war"
                        ssh -p 2222 -o StrictHostKeyChecking=no root@10.168.133.22 "systemctl restart tomcat"
                    """
                }
            }
        }
    }
}
