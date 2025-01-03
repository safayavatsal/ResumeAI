# ResumeAI

## Overview

The project is a MEAN project and uses node version 18.

# Run project locally

* Clone the repository in your local system.
* Created a .env file for environment variables
```
MONGO_URL="mongodb+srv://**************/resume_builder"
JWT_SECRET_KEY="MYREALLYSECRETKEY"
OPENAI_KEY="OPENAI_API_KEY"
GMAIL_USER="THIS EMAIL IS USED TO SEND RESUMES"
GMAIL_PASS="PASSWORD USED BY NODEMAILER"
FRONT_END="URL FOR FRONTEND"
```
* Go to ResumeBuilderBackend folder and install the dependencies mentioned below
```
npm install
npm install -g typescript
tsc --build
```
*  After installing build the application.
```
npm run start
```
* Test the localhost:4292 for an output
```
Cannot GET /
```
![alt text](images/image.png)
* Then install dependencies in frontend move to ResumeBuilderAngular and run the commands mentioned below
```
npm install -g @angular/cli
npm install -g angular-http-server
npm install --force
```
* Run the application 
```
npm start
```
* Test the localhost:4200 for an output
![alt text](images/image1.png)

Congrats your local set up is done next will procede with Dockerfile

# Create a Dockerfile and push it into ECR

# Backend deployment

* Cloning the project .env file updatation are already done lets procede with writing docker file.
* Creat a Dockerfile with defined node version, dependencies, dist file creation and command to run application application.
* Dockerfile is updated in in ResumeBuilderBckend folder.
* Then build and test docker file locally with the help of commads mentioned below.
```
docker build -t <docker image name>
docker run -p 4292:4292 <image name>
```
* Test the localhost:4292 for an output
```
Cannot GET /
```
* push the image into docker hub and amazon ECR
* Create a new repository and pushed image into AWS ECR using the commands mentioned below.
```
# create a repository
aws ecr create-repository --repository-name <name> --region <region>

# login connection 
aws ecr get-login-password --region <region> | docker login --username <AWS> --password-stdin <aws_account_id.dkr.ecr.region.amazonaws.com>

#docker tag 
docker tag  image <e9ae3c220b23 aws_account_id.dkr.ecr.region.amazonaws.com / my-repository:tag>

# push image

docker push aws_account_id.dkr.ecr.region.amazonaws.com/my-repository:tag

```
######### well done your backend image successfully pushed in ECR #####

# Frontend deployment

* Repeat the steps similar as backend deployment make sure the port changed into 4200, other steps are more or less similar.

####### congrats frontend also deployed ######

# Docker compose.

* creat a docker compose file with yaml extension
* Backend service is build on port 4292 and frontend file is build on port 4200
* compose file is mentioned in the name of docker-compose.yaml.
* using docker-compose up command to build the images and deploy.

#####Cocker compose also done ########

# Kubernetes cluster creation and deploy the application in local lost 

* Create a Kubernetes deployment file with yaml extension.
* Go to ResumebuilderBackend folder and create deploymeny.yml file, deployment file are the one which manage pods, services and the entire eco system.
* deployment.yml file contains number of replicas, image need to deploy and the port where the deployment going to happened.
* Service.yaml file defines the service name, port.
* command used to deploy deployment and service file 
```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```
![alt text](images/image2.PNG)

```
# get number of deployments 
kubectl get deployments
```
![alt text](images/image3.PNG)

```
# get number of pods that are running 
kubectl get pods
```
![alt text](images/image4.PNG)

```
# get service details
kubectl get svc
```
![alt text](images/image5.PNG)

To view the Minikube deployment dashboard, use the following command:
```
minikube dashboard
```
The code displays the following in your web browser:
![alt text](images/image6.PNG)

To expose the service, use the following `minikube` command:
```
minikube service resumebuilderbackend-evc
```

![alt text](images/image7.PNG)

open the url in browser to see the result

![alt text](images/image7.PNG)

######## well done backend deployment was done using kubernetes #########