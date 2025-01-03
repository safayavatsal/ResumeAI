pipeline {
    agent any
    environment{
        AWS_ACCESS_KEY='IAM_Poo'
        SSH_KEY = credentials('ssh_poo')
        DOCKERHUB_CREDENTIALS= 'docker_poo'
        DOCKER_IMAGE_RESUME_BUILDER_FRONTEND = 'flowerking21/resume_fe'
        DOCKER_IMAGE_RESUME_BUILDER_BACKEND = 'flowerking21/resume_be'
       
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
        stage('CHECKOUT') {
            steps {
                echo 'clone the git code' 
                git branch: 'main', url:'https://github.com/flowerpoo/Resume_AI.git'
            }
        }
        
        stage('create .env'){
            steps{
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
        
        
        stage('build images') {
            parallel {
                stage('build backend') {
                    steps {
                        script {
                            docker.build("${env.DOCKER_IMAGE_RESUME_BUILDER_BACKEND}:${env.BUILD_ID}", './ResumeBuilderBackend/')
                            //echo ("done")
                        }
                    }
                }
                stage('build frontend') {
                    steps {
                        script {
                            docker.build("${env.DOCKER_IMAGE_RESUME_BUILDER_FRONTEND}:${env.BUILD_ID}", './ResumeBuilderAngular/')
                            // echo ("done")
                        }
                    }
                }
            }
        }
        
        stage('push to docker'){
            steps{
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                       docker.image("${env.DOCKER_IMAGE_RESUME_BUILDER_BACKEND}:${env.BUILD_ID}").push()
                       docker.image("${env.DOCKER_IMAGE_RESUME_BUILDER_FRONTEND}:${env.BUILD_ID}").push()
                       //echo ("done")
                    }
                }
            }
        }
        
        stage('eks connection'){
            steps{
                script{
                     withCredentials([aws(credentialsId: 'IAM_Poo', region:'eu-west-2')]) {
                        echo "login success"
                        def eksClusterExists = sh(script: 'aws eks describe-cluster --name poo-eks-cluster-1 --region eu-west-2', 
                        returnStatus: true) == 0
                        if(!eksClusterExists)
                        {
                            sh '''
                            eksctl create cluster --name poo-eks-cluster-1 --region eu-west-2 --nodegroup-name standard-workers --node-type t2.micro --nodes 2 --nodes-min 1 --nodes-max 3
                            '''
                        }
                        else{
                             sh '''
                            kubectl version --client
                       
                           aws eks --region eu-west-2 update-kubeconfig --name poo-eks-cluster-1
                        
                          
                           cd  ResumeBuilderAngular/
                           kubectl apply -f frontend-deployment.yaml
                           kubectl apply -f backend-service.yaml
                          '''
                            
                        }
                        
                       
                        
                    }
                }
            }
        }
        
        // stage("eks deployment"){
        //     steps{
        //         script{
        //             withCredentials([aws(credentialsId: 'IAM_Poo', region:'eu-west-2')]) {
        //                 sh '''
        //                 kubectl version --client
                       
        //                 aws eks --region eu-west-2 update-kubeconfig --name poo-eks-cluster-1
                        
        //                 cd ResumeBuilderBackend/
        //                 kubectl apply -f backend-deployment.yaml
        //                 kubectl apply -f backend-service.yaml
        //                 '''
        //             }
        //         }
        //     }
        // }
        
    }
}
