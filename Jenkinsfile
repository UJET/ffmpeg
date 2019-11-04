def docker_image
def image_tag
def identity

pipeline {
    agent any

    parameters {
        string(defaultValue: 'us-west-2', description: 'Region where we run the pipeline', name: 'REGION')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup') {
            steps {
                script {
                    identity = awsIdentity()
                }
            }
        }


        stage('Build Image') {
            steps {
                echo 'About to build a new docker image for ffmpeg-static'
                dir ('docker-images') {
                    script {
                        docker_image = docker.build("ffmpeg-static", "-f 4.2/alpine/Dockerfile .")
                    }
                }
            }
        }

        stage('Push docker image') {
            steps {
                echo 'Pushing docker image to the remote repository'
                script {
                    image_tag = "${identity.account}.dkr.ecr.${REGION}.amazonaws.com/ffmpeg-static"
                    sh ecrLogin()
                    docker.withRegistry("https://${image_tag}") {
                        docker_image.push("4.2-alpine")
                    }
                }
            }
        }
    }

    post {
        always {
            deleteDir() /* clean up our workspace */
        }
        success {
            echo 'Build succeeded!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
