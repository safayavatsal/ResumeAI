pipeline {
    agent any
    environment {
        GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp-service-account-key')
        SSH_KEY = credentials('ssh_poo')
        DOCKERHUB_CREDENTIALS = 'docker_poo'
        DOCKER_IMAGE_RESUME_BUILDER_FRONTEND = 'flowerking21/resume_fe'
        DOCKER_IMAGE_RESUME_BUILDER_BACKEND = 'flowerking21/resume_be'
        GCP_PROJECT = 'your-gcp-project-id'
        GKE_CLUSTER = 'poo-gke-cluster'
        GKE_REGION = 'us-central1'
        KUBECONFIG_PATH = '/tmp/kubeconfig'
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
        stage('CHECKOUT') {
            steps {
                echo 'Cloning the Git repository'
                git branch: 'main', url: 'https://github.com/flowerpoo/Resume_AI.git'
            }
        }

        stage('Create .env') {
            steps {
                script {
                    def envContent = """
                        MONGO_URL='mongodb+srv://**************/resume_builder'
                        JWT_SECRET_KEY="MYREALLYSECRETKEY"
                        OPENAI_KEY="OPENAI_API_KEY"
                        GMAIL_USER="THIS EMAIL IS USED TO SEND RESUMES"
                        GMAIL_PASS="PASSWORD USED BY NODEMAILER"
                        FRONT_END="http://localhost:4292"
                    """
                    writeFile(file: './ResumeBuilderBackend/.env', text: envContent.trim())
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Backend') {
                    steps {
                        script {
                            docker.build("${env.DOCKER_IMAGE_RESUME_BUILDER_BACKEND}:${env.BUILD_ID}", './ResumeBuilderBackend/')
                        }
                    }
                }
                stage('Build Frontend') {
                    steps {
                        script {
                            docker.build("${env.DOCKER_IMAGE_RESUME_BUILDER_FRONTEND}:${env.BUILD_ID}", './ResumeBuilderAngular/')
                        }
                    }
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        docker.image("${env.DOCKER_IMAGE_RESUME_BUILDER_BACKEND}:${env.BUILD_ID}").push()
                        docker.image("${env.DOCKER_IMAGE_RESUME_BUILDER_FRONTEND}:${env.BUILD_ID}").push()
                    }
                }
            }
        }

        stage('GKE Connection and Deployment') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        // Authenticate and configure GKE
                        sh '''
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud config set project ${GCP_PROJECT}
                        gcloud container clusters get-credentials ${GKE_CLUSTER} --region ${GKE_REGION} --kubeconfig=${KUBECONFIG_PATH}
                        export KUBECONFIG=${KUBECONFIG_PATH}
                        '''

                        // Apply Kubernetes configurations
                        sh '''
                        kubectl apply -f ResumeBuilderAngular/frontend-deployment.yaml
                        kubectl apply -f ResumeBuilderAngular/backend-service.yaml
                        kubectl apply -f ResumeBuilderBackend/backend-deployment.yaml
                        kubectl apply -f ResumeBuilderBackend/backend-service.yaml
                        '''
                    }
                }
            }
        }
    }
}
