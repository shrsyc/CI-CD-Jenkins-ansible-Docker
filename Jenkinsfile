pipeline {
    agent any
    tools {
        maven 'maven'
    }
    stages {
        stage("Pull SRC") {
            steps {
                git 'https://github.com/shrsyc/instagram-war.git'
            }
        }
        stage("Prepare Build") {
            steps {
                sh 'mvn clean package'
            }
        }
        stage("Copy *.war file to ansible") {
            steps {
                sh 'mv target/instagram.war .'
                sshPublisher(
                    continueOnError: false, 
                    failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: "marcos",
                            transfers: [sshTransfer(sourceFiles: 'instagram.war')],
                            verbose: true
                        )
                    ]
                )
            }
        }
        stage("Copy Docker file to ansible to push docker immage to dockerhub") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                sshPublisher(
                    continueOnError: false, 
                    failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: "marcos",
                            transfers: [
                                sshTransfer(sourceFiles: 'Dockerfile'),
                                sshTransfer(execCommand: """
                                docker rm -f tomcat || true
                                docker rmi -f $DOCKER_USER/webapp_portfolio || true
                                docker build -t $DOCKER_USER/webapp_portfolio .
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                                docker push $DOCKER_USER/webapp_portfolio
                                """)
                            ],
                            verbose: true
                        )
                    ]
                )
            }
            }
        }
        stage("Copy playbook file to ansible and execute") {
            steps {
               
                sshPublisher(
                    continueOnError: false, 
                    failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: "marcos",
                            transfers: [
                                sshTransfer(sourceFiles: 'ansible_playbook.yml'),
                                sshTransfer(execCommand: "ansible-playbook ansible_playbook.yml")
                            ],
                            verbose: true
                        )
                    ]
                )
            
            }
        }
    }
}
