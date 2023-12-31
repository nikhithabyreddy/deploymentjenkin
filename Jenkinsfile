pipeline {
    agent any

    parameters {
        string(name: 'PARAMETER_NAME', defaultValue: 'default_value', description: 'Parameter description')
        booleanParam(name: 'ENABLE_FEATURE', defaultValue: true, description: 'Enable feature flag')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'prod'], description: 'Select deployment environment')
    }

    environment {
        function_name = 'jenkins'
        AWS_REGION = 'us-east-2'
        S3_BUCKET = 'jenkinsbucket28'
        S3_KEY = 'sample-1.0.3.jar'
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
                sh "aws lambda update-function-code --function-name $function_name --region $AWS_REGION --s3-bucket $S3_BUCKET --s3-key $S3_KEY"
            }
        }
        
        stage('Production Confirmation') {
            when {
                expression { params.ENVIRONMENT == 'prod' }
            }
            steps {
                echo 'Are you ready for production?'
                // Input step highlighted below
                input(message: 'Are you ready for production?', ok: 'Proceed to production')
            }
        }
    }

    post {
        always {
            echo 'This step will always execute'
            echo "${env.BUILD_ID}"
            echo "${BRANCH_NAME}"
            echo "${JENKINS_URL}"
        }
        success {
            echo 'This step will only execute on successful builds'
            emailext subject: "Jenkins Build Success",
                      body: "The Jenkins build was successful. You can find the artifacts at <insert artifact location>",
                      to: "nikithareddykondam@gmail.com"
        }
        failure {
            echo 'This step will only execute on failed builds'
            emailext subject: "Jenkins Build Failed",
                      body: "The Jenkins build failed. Please investigate the issue.",
                      to: "nikithareddykondam@gmail.com"
        }
    }
}






