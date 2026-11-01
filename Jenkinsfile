def run(cmd) {
    if (isUnix()) {
        sh cmd
    } else {
        bat cmd
    }
}

def awsRun(cmd) {
    withCredentials([
        string(credentialsId: 'aws-key-id',
               variable: 'AWS_ACCESS_KEY_ID'),
        string(credentialsId: 'aws-key-id',
               variable: 'AWS_SECRET_ACCESS_KEY')
    ]) {
        run(cmd)
    }
}

def ecrLogin() {
    awsRun "aws ecr get-login-password --region ${env.REGION} | docker login --username AWS --password-stdin ${env.REGISTRY}"
}

def deploy() {
    awsRun "aws ssm send-command --region ${env.REGION} --instance-ids ${env.INSTANCE} --document-name AWS-RunShellScript --parameters file://deploy.json"
}

pipeline {

    agent any

    environment {
         PATH     = "C:\\Users\\baisk\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;${PATH}"
        REGION   = 'ap-south-1'

        ACCOUNT  = '754928766311'

        REPO     = 'hello-jenkins'

        INSTANCE = 'none'

        REGISTRY = "${ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com"

        IMG      = "${REPO}:${BUILD_NUMBER}"

        REMOTE   = "${REGISTRY}/${REPO}:latest"
    }

    stages {

        stage('Build') {

            steps {

                script {

                    run "docker build -t ${IMG} ."

                }
            }
        }

        stage('Test') {

            steps {

                script {

                    run "docker run --rm ${IMG} nginx -t"

                }
            }
        }

       stage('Push') {
    when { expression { false } }
    steps {

                script {

                    ecrLogin()

                    run "docker tag ${IMG} ${REMOTE}"

                    run "docker push ${REMOTE}"

                }
            }
        }

        stage('Deploy') {

            when {

                expression {
                    env.INSTANCE != 'none' &&
                    env.INSTANCE?.trim()
                }

            }

            steps {

                script {

                    deploy()

                }
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}
