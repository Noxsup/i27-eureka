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
        SONAR_URL = "http://35.232.164.117:9000"
        SONAR_TOKEN = credentials('sonar_creds')
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
                sh """
                    echo "Starting sonar scan"
                    mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=i27-Eureka \
                        -Dsonar.host.url=${env.SONAR_URL} \
                        -Dsonar.login=${env.SONAR_TOKEN}
                """
            }
        }
    }
    

}
