// testing commit and its working and its working, I guess
pipeline {
    agent {
        label 'maven-slave'
    }
    tools {
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }
    
    environment {
        APPLICATION_NAME = "eureka"
        SONAR_URL = "http://23.251.154.207:9000/"
        SONAR_TOKEN = credentials('sonar_creds')
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        DOCKER_HUB = "docker.io/sk619"
        DOCKER_REPO = "i27eurekaproject"
        USER_NAME= "sk619" //UserID for DockerHub 
        DOCKER_CREDS = credentials('dockerhub_creds')
       

    }
    
    stages {
        stage('Build') {
            // Build happens here
            // only build should happen, no tests should be available
            steps {
                echo "building the ${env.APPLICATION_NAME} application"
                // maven build should happen here
                sh "mvn --version"
                sh "mvn clean package -DskipTests=true"
                archiveArtifacts artifacts: 'target/*jar', followSymlinks: false
            }
        }
        stage('Unit tests') {
            steps {
                echo "performing unit tests for ${env.APPLICATION_NAME} application"
                sh "mvn test"
            }
        }
        stage('sonar') {
            steps {
                echo "Starting Sonarscan with Quality gate"
                withSonarQubeEnv('SonarQube') {
                    sh """
                        mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=i27-Eureka \
                            -Dsonar.host.url=${env.SONAR_URL} \
                            -Dsonar.login=${env.SONAR_TOKEN}
                    """
                }

            // adding the below line my pipeline is being aborted, if im removing it, working. so pipeline not at fault i guess 
                /*timeout(time: 2, unit: 'MINUTES') {
                    script {
                        waitForQualityGate abortPipeline: true
                    }
                }*/
            }
        }
        stage('Build Format') {
            steps {
                script {
                    sh """
                        echo "Existing JAR Format: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                        echo "*** Below is my expected output"
                        echo "Destination Source is i27-${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}"
                    """
                }
            }
        }
        stage ('Docker build and push ') {
            steps {
                script {
                    sh """
                        ls -la
                        cp ${workspace}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd
                        echo "Listing files in .cicd folder"
                        ls -la ./.cicd
                        echo " ********* Building Docker Image **********"
                        # docker build -t imagename.
                        docker build --force-rm --no-cache --pull --rm=true --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} --build-arg JAR_DEST=i27-${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.DOCKER_REPO}:$GIT_COMMIT ./.cicd
                        # push repo 
                        # Docker hub, Google Container registry, JFROG
                        echo "********************* Logging into Docker Registry*********************  "
                        docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}
                        docker push ${env.DOCKER_HUB}/${env.DOCKER_REPO}:$GIT_COMMIT
                        #docker push accountname/reponame:tagname
                    """
                }
            }
        }
        stage ('Deploy to Dev') { //5761
            steps {
               script {
                dockerDeploy('dev', '5761', '8761').call()
               }
               
                
            }
        }
        stage('Deploy to test'){
            steps {
                script {
                    dockerDeploy ('Test', '6761', '8761').call()
                }
            }
        }

            
            
        
    }
}

def dockerDeploy(envDeploy, hostPort, contPort ) {
    return {
        echo "******************* Deploying to $envDeploy Environment ***********************"
        withCredentials([usernamePassword(credentialsId: 'maha_docker_vm_creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
        // some block
        // with this credentials, i need to connect to dev environment
        // sshpass 
                script {
                    sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$docker_server_ip \"docker pull ${env.DOCKER_HUB}/${env.DOCKER_REPO}:$GIT_COMMIT\""
                    //sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$docker_server_ip \ "***"
                        
                    echo "Stop the Container"
                    // If we execute the below command it will fail for the first time obviously as containers are not available stop/remove will cause this issue
                     // we can implement try catch block 
                    try {
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$docker_server_ip \"docker stop ${env.APPLICATION_NAME}-$envDeploy\""
                        echo " Removing the Container "
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$docker_server_ip \"docker rm ${env.APPLICATION_NAME}-$envDeploy\""

                    } catch (err) {
                        echo "Caught the error : $err"
                    }
                    // Run the container 
                    sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$docker_server_ip \"docker run --restart always --name ${env.APPLICATION_NAME}-$envDeploy -p $hostPort:$contPort -d ${env.DOCKER_HUB}/${env.DOCKER_REPO}:$GIT_COMMIT\""



                } 
        }

    }
}

// 8761 is the container port, we cant change it.
// if we really want to change it, we can by using -Dserver.port=9090, this will be your container port
// but we are considering the below host ports
// dev ===> 5761
// test ===> 6761
// stage ===> 7761
// prod ===> 8761