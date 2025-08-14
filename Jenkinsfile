pipeline {
    agent any

    tools {
        maven 'Maven3' 
        jdk 'JDK17'
    }

    environment {
        SONARQUBE_ENV = 'LocalSonar' 
        JFROG_SERVER_ID = 'artifactory-server' 
        JFROG_REPO = 'my-repo-local'          
        COUNTER_FILE = "${WORKSPACE}/counter.txt"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/LokesYadav/maven-project.git'
            }
        }
         stage('Update Counter') {
            steps {
                script {
                    if (fileExists(env.COUNTER_FILE)) {
                        counter = readFile(env.COUNTER_FILE).trim().toInteger() + 1
                    } else {
                        counter = 1
                    }
                    writeFile file: env.COUNTER_FILE, text: counter.toString()
                    echo "🔢 Job run counter: ${counter}"
                    env.JOB_RUN_COUNTER = counter.toString()
                }
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
                sh "mvn package -Djar.finalName=maven-project-${env.JOB_RUN_COUNTER}"
            }
        }


       stage('Upload to JFrog') {
            steps {
                rtUpload (
                    serverId: "${JFROG_SERVER_ID}",
                    spec: """{
                        "files": [
                            {
                                "pattern": "target/maven-project-${env.JOB_RUN_COUNTER}.jar",
                                "target": "${JFROG_REPO}/"
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

