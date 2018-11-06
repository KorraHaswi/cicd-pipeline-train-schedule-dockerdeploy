pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
     stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {*
                    app = docker.build("korrahaswi/train-schedule") /*app variable is a referene to the docker
                                                                     image that we built here using the script 
                                                                     style instead of declarative stype*/
					app.inside{ //app.inside mean to run the below command inside the image
                    app.inside {   //app.inside mean to run the below command inside the image
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {  /*here we are using the docker
                                                                                                    website registry instead of private 
                                                                                                    one. credentials used are the the 
                                                                                                    one we created for the docker hub
                                                                                                    in jenkins*/
                        app.push("${env.BUILD_NUMBER}") // a tag to trakc the image being pushed. app variable is a reference to that image
                        app.push("latest") /*this tag mentios that it is latest.after build docker and push chekc docker.com
                                           and see your code theri with build number as well as latest tag*/
                    }
                }
            }
        }
    
    
  stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) { /*using these creds to access the 
                                                                                                                                                     prod server*/                               
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull korrahaswi/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d korrahaswi/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
    
 }
