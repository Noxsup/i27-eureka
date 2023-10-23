// testing commit
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
    }
}
