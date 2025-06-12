## How to execute this pipeline

1) Sonarscanner
2) Sonarqube server
3) jenkins server
4) nexus server
5) change the credetilas of nexus server as per our requirement
6) create a new repo in nexus with version policy as "snapshot"
7) modiy the pipeline as per our sonarscanner name and sonarqube server name

### Use the below code in sonarqube analysis report.

```xml
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
```
