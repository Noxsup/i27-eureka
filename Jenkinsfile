// test file
pipeline {
    agent any
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
                sh "mvn clean package -DskipTests=true"
            }
        }
    }
}
