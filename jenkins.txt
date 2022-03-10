pipeline {
    agent any
    environment {
        registry = "987240292938.dkr.ecr.ap-south-1.amazonaws.com/http"
    }
    stages {
        stage ('Checkout') {
            steps {
            checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/theteprasad/ecs.git']]])
        }
        }
        
            stage ('Docker Build') {
                steps {
                    script {
                            dockerImage = docker.build registry
                    }
                }
            }
            stage ('Push Docker image') {
            steps {
                sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 987240292938.dkr.ecr.ap-south-1.amazonaws.com'
                sh 'docker push 987240292938.dkr.ecr.ap-south-1.amazonaws.com/http:latest'
  }
}
            stage ('Saving the docker image in tar') {
                steps {
                    sh 'docker image save -o apache.tar 987240292938.dkr.ecr.ap-south-1.amazonaws.com/http:latest'
                }
            }
            
            stage ('Upload the image file in s3') {
                steps {
                    sh 'aws s3 cp /var/lib/jenkins/workspace/new/apache.tar s3://jenkins1010'
                }
            }
            
            stage ('upload appspec and other files to s3') {
                steps {
                    sh 'tar -cvf app.tar scripts/'
                    sh 'aws s3 cp /var/lib/jenkins/workspace/new/app.tar s3://jenkins1010'
                    sh 'aws s3 cp appspec.yml s3://jenkins1010'
                }
            }
            }
}
        
