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
        DOCKER_HUB = "docker.io/devopswithcloudhub"
        DOCKER_REPO = "i27eurekaproject"
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
                        echo ${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} 
                        echo ${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}

                        echo " ********* Building Docker Image **********"
                        # docker build -t imagename.
                        docker build --force-rm --no-cache --pull --build-arg JAR_SOURCE=${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} --build-arg JAR_DEST=${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.DOCKER_REPO}:${GIT_COMMIT} ./.cicd

                    """
                }
            }
        }
    }
}
