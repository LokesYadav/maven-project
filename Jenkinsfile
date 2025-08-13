// pipeline {
//     agent any

//     tools {
//         maven 'Maven3' 
//         jdk 'JDK17'
//     }

//     environment {
//         SONARQUBE_ENV = 'LocalSonar' // Jenkins → Manage Jenkins → Configure System
//         JFROG_SERVER_ID = 'artifactory-server' // Must match Jenkins Artifactory config
//         JFROG_REPO = 'my-repo-local'           // Name of local repo in JFrog
//     }

//     stages {
//         stage('Checkout') {
//             steps {
//                 git branch: 'main', url: 'https://github.com/LokesYadav/maven-project.git'
//             }
//         }

//         stage('Build') {
//             steps {
//                 sh 'mvn clean compile'
//             }
//         }

//         stage('Test') {
//             steps {
//                 sh 'mvn test'
//             }
//         }

//         stage('SonarQube Analysis') {
//             steps {
//                 withSonarQubeEnv("${SONARQUBE_ENV}") {
//                     sh '''
//                         mvn sonar:sonar \
//                         -Dsonar.projectKey=maven-project \
//                         -Dsonar.projectName="Maven Project" \
//                         -Dsonar.host.url=http://localhost:9000 \
//                         -Dsonar.login=${SONARQUBE_AUTH_TOKEN1}
//                     '''
//                 }
//             }
//         }

//         stage('Package') {
//             steps {
//                 sh 'mvn package'
//             }
//         }

//         stage('Upload to JFrog') {
//             steps {
//                 rtUpload (
//                     serverId: "${JFROG_SERVER_ID}",
//                     spec: """{
//                         "files": [
//                             {
//                                 "pattern": "target/*.jar",
//                                 "target": "${JFROG_REPO}/"
//                             }
//                         ]
//                     }"""
//                 )
//             }
//         }

//         stage('Publish Build Info') {
//             steps {
//                 rtPublishBuildInfo(
//                     serverId: "${JFROG_SERVER_ID}"
//                 )
//             }
//         }

//         stage('Deploy via Ansible') {
//             steps {
//                 echo "🚀 Starting deployment using Ansible..."
//                 sh """
//                     ansible-playbook -i ${INVENTORY_FILE} ${PLAYBOOK_FILE} \
//                     -u ${ANSIBLE_USER} --private-key /home/jenkins/.ssh/id_rsa
//                 """
//             }
//         }
//     }

//     post {
//         success {
//             echo '✅ Build and upload to JFrog completed successfully!'
//         }
//         failure {
//             echo '❌ Build failed!'
//         }
//     }
// }


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
        INVENTORY_FILE = '/home/lkumar/playbooks/inventory.ini'   // ✅ Set your inventory path here
        PLAYBOOK_FILE = '/home/lkumar/playbooks/deploy_app.yml'  // ✅ Set your playbook path
        SSH_USER = 'lkumar'                                      // SSH username for target servers
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

        stage('Upload to JFrog') {
            steps {
                rtUpload (
                    serverId: "${JFROG_SERVER_ID}",
                    spec: """{
                        "files": [
                            {
                                "pattern": "target/*.jar",
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

        stage('Deploy via Ansible') {
            steps {
                echo "🚀 Starting deployment using Ansible..."
                sh """
                    ansible-playbook -i ${INVENTORY_FILE} ${PLAYBOOK_FILE} -u ${SSH_USER} -k
                """
            }
        }
    }

    post {
        success {
            echo '✅ Build, upload, and deployment completed successfully!'
        }
        failure {
            echo '❌ Build or deployment failed!'
        }
    }
}

