// testing commit and its working and its working i guess
pipeline {
    agent {
        label 'maven-slave'
    }
    tools{
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }
    
    environment {
        APPLICATION_NAME = "eureka"
        SONAR_URL = "http://23.251.154.207:9000/"
        SONAR_TOKEN = credentials('sonar_creds')
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
    }
    
    stages{
        stage('Build'){
            // Build happens here
            // only build should happen, no tests should be available
            steps{
                echo "building the ${env.APPLICATION_NAME} application"
                // maven build should happen here
                sh "mvn --version"
                sh "mvn clean package -DskipTests=true"
                archiveArtifacts artifacts: 'target/*jar', followSymlinks: false
            }
        }
        stage ('Unit tests'){
            steps {
                echo "performing unit tests for ${env.APPLICATION_NAME} application "
                sh "mvn test"
            }
        }
        stage ('sonar') {
            steps {
                echo "Starting Sonarscan with Quality gate"
                withSonarQubeEnv('SonarQube'){
                    sh """
                        mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=i27-Eureka \
                            -Dsonar.host.url=${env.SONAR_URL} \
                            -Dsonar.login=${env.SONAR_TOKEN}
                    
                """

                }
                timeout (time: 2, unit: 'MINUTES' ) { //NANOSECONDS, MINUTES ....
                    script {
                        waitForQualityGate abortPipeline: false
                    }

                }
            }
        }
        stage('Build Format') { // i27-eureka-0.0.1-SNAPSHOT.jar
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
    }
    

}

