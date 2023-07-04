pipeline {
    agent any

    parameters {
        string(name: 'PARAMETER_NAME', defaultValue: 'default_value', description: 'Parameter description')
        booleanParam(name: 'ENABLE_FEATURE', defaultValue: true, description: 'Enable feature flag')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'prod'], description: 'Select deployment environment')
    }

    environment {
        function_name = 'jenkins'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Build'
                sh 'mvn package'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    // Run SonarQube analysis
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Push') {
            steps {
                echo 'Push'
                sh "aws s3 cp target/sample-1.0.3.jar s3://jenkinsbucket28"
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploy'
                sh "aws lambda update-function-code --function-name $function_name --region us-east-2 --s3-bucket jenkinsbucket28 --s3-key sample-1.0.3.jar"
            }
        }
        
        stage('Production Confirmation') {
            steps {
                 when {
                    expression { params.ENVIRONMENT == 'prod' }
                }
                echo 'Are you ready for production?'
                // Input step highlighted below
                input(message: 'Are you ready for production?', ok: 'Proceed to production')
            }
        }
    }

    post {
        always {
            echo 'This step will always execute'
        }
        success {
            echo 'This step will only execute on successful builds'
            emailext subject: "Jenkins Build Success",
                      body: "The Jenkins build was successful. You can find the artifacts at <insert artifact location>",
                      to: "nikireddy2109@gmail.com"
        }
        failure {
            echo 'This step will only execute on failed builds'
            emailext subject: "Jenkins Build Failed",
                      body: "The Jenkins build failed. Please investigate the issue.",
                      to: "nikireddy2109@gmail.com"
        }
    }
}



