pipeline {
    agent any

    tools {
        maven 'maven-3.9.9'
        jdk 'Open-JDK-21'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/Nitesh8011/java_pipeline.git']]
                ])
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv(installationName: 'SonarCloud', credentialsId: 'SONARCLOUDTOKEN') {
                    sh """
                        mvn clean test verify jacoco:report \
                        org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                        -Dgpg.skip=true \
                        -Dsonar.organization='nitesh-kumar-personal' \
                        -Dsonar.host.url='https://sonarcloud.io' \
                        -Dsonar.projectKey='nitesh-kumar-personal_java21-spring-boot3-rest-api-archetype' \
                        -Dproject.build.sourceEncoding=UTF-8
                    """
                }
                publishCoverage adapters: [
                    jacocoAdapter(
                        path: 'target/site/jacoco/jacoco.xml'
                    )
                ]
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Cyclomatic Complexity') {
            steps {
                sh """
                    wget https://bootstrap.pypa.io/get-pip.py
                    python3 get-pip.py --user
                    export PATH=\$HOME/.local/bin:\$PATH
                    pip install --user lizard
                    lizard -l java src/
                """
            }
        }

        stage('OWASP Dependency-Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format ALL', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/dependency-check-report.html', fingerprint: true
            emailext (
                to: 'nitesh20311@gmail.com',
                subject: "Build ${currentBuild.result ?: 'SUCCESS'} - ${env.JOB_NAME}",
                mimeType: 'text/html',
                body: """
                    <html>
                    <body>
                        <h2>Jenkins Build Notification</h2>
                        <p><strong>Project:</strong> ${env.JOB_NAME}</p>
                        <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                        <p><strong>Status:</strong> <span style="color: ${currentBuild.result == 'SUCCESS' ? 'green' : 'red'};">${currentBuild.result ?: 'SUCCESS'}</span></p>
                        <p>Check details at: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                    </body>
                    </html>
                """
            )
        }
    }
}