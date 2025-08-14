pipeline {
    agent any

    tools {
        maven 'Maven3' 
        jdk 'JDK17'
    }

    environment {
        SONARQUBE_ENV = 'LocalSonar' // Jenkins → Manage Jenkins → Configure System
        JFROG_SERVER_ID = 'artifactory-server' // Must match Jenkins Artifactory config
        JFROG_REPO = 'my-repo-local'           // Name of local repo in JFrog
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/LokesYadav/maven-project.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=maven-project \
                        -Dsonar.projectName="Maven Project" \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=${SONARQUBE_AUTH_TOKEN1}
                    '''
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Rename Artifact with Build Number') {
            steps {
                sh '''
                    ARTIFACT=$(ls target/*.jar | head -n 1)
                    BASENAME=$(basename "$ARTIFACT" .jar)
                '''
            }
        }

        stage('Upload to JFrog') {
            steps {
                rtUpload (
                    serverId: "${JFROG_SERVER_ID}",
                    spec: """{
                        "files": [
                            {
                                "pattern": "target/*-${BUILD_NUMBER+11}.jar",
                                "target": "${JFROG_REPO}/${env.JOB_NAME}/${env.BUILD_NUMBER}/"
                            }
                        ]
                    }"""
                )
            }
        }

        stage('Publish Build Info') {
            steps {
                rtPublishBuildInfo(
                    serverId: "${JFROG_SERVER_ID}"
                )
            }
        }
    }

//     post {
//         success {
//             echo '✅ Build and upload to JFrog completed successfully!'
//         }
//         failure {
//             echo '❌ Build failed!'
//         }
//     }
// }

    post {
        success {
            mail to: 'lokeshydv20@gmail.com',
                 subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Good news! The build succeeded. Check details here: ${env.BUILD_URL}"
            echo '✅ Build and upload to JFrog completed successfully!'
        }
        failure {
            mail to: 'lokeshydv20@gmail.com',
                 subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Oops! The build failed. Check details here: ${env.BUILD_URL}"
            echo '❌ Build failed!'
        }
    }
}

