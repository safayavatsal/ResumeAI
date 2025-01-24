pipeline {
    agent any
    environment {
        GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp-service-account-key')
        SSH_KEY = credentials('ssh_poo')
        DOCKERHUB_CREDENTIALS = 'docker_poo'
        DOCKER_IMAGE_RESUME_BUILDER_FRONTEND = 'gcr.io/flowerking21/resume_fe'
        DOCKER_IMAGE_RESUME_BUILDER_BACKEND = 'gcr.io/flowerking21/resume_be'
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
                git branch: 'test-2', url: 'https://github.com/safayavatsal/Resume_AI.git'
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
        
        stage('Push to Google Container Registry') {
            steps {
                script {
                    sh """
                    docker tag ${env.DOCKER_IMAGE_RESUME_BUILDER_BACKEND}:${env.BUILD_ID} gcr.io/${env.GCP_PROJECT}/resume-be:${env.BUILD_ID}
                    docker tag ${env.DOCKER_IMAGE_RESUME_BUILDER_FRONTEND}:${env.BUILD_ID} gcr.io/${env.GCP_PROJECT}/resume-fe:${env.BUILD_ID}
                    docker push gcr.io/${env.GCP_PROJECT}/resume-be:${env.BUILD_ID}
                    docker push gcr.io/${env.GCP_PROJECT}/resume-fe:${env.BUILD_ID}
                    """
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

                        // Update Kubernetes YAML with dynamic Docker image tags
                        sh '''
                        sed -i "s|gcr.io/.*/resume-be:.*|gcr.io/${GCP_PROJECT}/resume-be:${BUILD_ID}|" ResumeBuilderBackend/backend-deployment.yaml
                        sed -i "s|gcr.io/.*/resume-fe:.*|gcr.io/${GCP_PROJECT}/resume-fe:${BUILD_ID}|" ResumeBuilderAngular/frontend-deployment.yaml
                        '''

                        // Apply Kubernetes configurations
                        sh '''
                        kubectl apply -f ResumeBuilderAngular/frontend-deployment.yaml
                        kubectl apply -f ResumeBuilderAngular/frontend-service.yaml
                        kubectl apply -f ResumeBuilderBackend/backend-deployment.yaml
                        kubectl apply -f ResumeBuilderBackend/backend-service.yaml
                        '''
                    }
                }
            }
        }
    }
}
