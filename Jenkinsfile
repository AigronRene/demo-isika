pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                echo '-=- Checkout project -=-'
                git branch: 'master', url:'https://github.com/aigronrene/demo-isika.git'
            }
        }
         stage('Compile') {
            steps {
                echo '-=- Compile project -=-'
                sh 'mvn clean compile'
            }
        }
        stage('Test') {
            steps {
                echo '-=- Test project -=-'
                sh 'mvn test'
            }
            post {
                success {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Package') {
            steps {
                echo '-=- Package project -=-'
                sh 'mvn package -DskipTests'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
        stage('Transfer file') {
            steps {
                script {
                    sshPublisher(publishers: [
                        sshPublisherDesc(configName: 'docker-host', transfers:[
                            sshTransfer(
                              sourceFiles:"target/*.jar",
                              removePrefixe:"target",
                              remoteDirectory:"//home//vagrant",
                              execCommand: "ls /"
                            ),
                            sshTransfer(
                              sourceFiles:"Dockerfile",
                              removePrefixe:"",
                              remoteDirectory:"//home//vagrant",
                              execCommand: '''
                                    cd /home/vagrant;
                                    sudo docker build -t demo-isika .;
                                    sudo docker run -d --name demo-isika -p 8080:8080 demo-isika;
                              '''
                            )
                        ])
                    ])
                }
            }
        }
        stage('Clean') {
            steps {
                echo '-=- Clean docker images & container -=-'
                sh 'ssh -v -o StrictHostKeyChecking=no vagrant@192.168.33.20 sudo docker stop demo-isika || true'
                sh 'ssh -v -o StrictHostKeyChecking=no vagrant@192.168.33.20 sudo docker rm demo-isika || true'
                sh 'ssh -v -o StrictHostKeyChecking=no vagrant@192.168.33.20 sudo docker rmi demo-isika || true'
            }
        }        
        stage('Construct image') {
            steps {
                echo '-=- Build image -=-'
                sh 'ssh -v -o StrictHostKeyChecking=no vagrant@192.168.33.20 sudo docker build -t demo-isika .'
            }
        }
        stage('Run container') {
            steps {
                echo '-=- Run container -=-'
                sh 'ssh -v -o StrictHostKeyChecking=no vagrant@192.168.33.20 sudo docker run -d --name demo-isika -p 8080:8080 demo-isika'
            }
        }
    }
}