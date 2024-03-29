# Task Management Web Application

- Used React for frontend.
- Wrote backend in Flask hooked to a SQLAlchemy database.
- Enabled commnunication between the two using REST APIs, CORS, and integrated JWT for authentication.
- Wrote dockerfiles, built images, and pushed to AWS ECR.
- Method 1: Deployed both containers individually using docker run | Method 2: Used AWS Elastic Kubernetes Service. -> Did both methods.
- Incorporated Terraform to set up Infrastructure as Code.
- Documented detailed approach in README.md file.
- Wrote GitLab CI/CD pipeline in an intermediary step to deploy the app using docker run | WIP: Deploying it on EKS + Terraform
  
___

**Notes (Read before you start):**
- Note 0: I have written this documentation as and when I was working on the project so the way of writing and what I want to cover will keep changing till I do a revamp of the entire thing. So bear with the inconsistent guiding methods.
- Note 1: I am a beginner myself, but if you're even greener than me/not getting any of the terms, I URGE you to open chatGPT and understand these terms instead of glossing over it. 
- Note 2: You can refer to my learnings towards the end of this readme to help you out of tough spots which I found myself in, there's a good chance you will come across those issues too.
- Note 3: Make sure you to understand what can start costing in your AWS using billing alerts. (EC2 instances + Cluster + Elastic IPs etc)
- Note 4: Dockers on your local can start consuming space if not cleaned regularly. 
- Note 5: For Approach 1 and 2, make the react code fetch from flask using localhost:5000 (the code may already be set as this or you need to in the fetch functions). For Approach 3, change it to load balancer DNS.


**How the app works:** 
- There is page for login + register, and another for adding tasks (homepage)
- Add tasks, refresh the page to see the added tasks (marked as Issue 1 under issues heading). 
- You can click on the tasks to move them around.
- When a task reaches 'Done' state and is clicked again, it deletes it in the backend but on the frontend, it pushes it in a cylical process back to 'To Do' (bug). More below.
- You have to refresh the page to see the task as deleted (marked as Issue 2, linked to Issue 1 I think).

## Running The App
There are several approaches you can take. I would suggest doing it using each approach one after the other to get a good understanding of the project. 

***Approach 1 (Local):*** Steps to Run the Full-Stack App Locally

- Backend: On one terminal, navigate to the folder named backend, and execute the command 'python3 app.py'  
- Frontend: On second terminal, navigate to the folder named frontend, and execute the command 'npm start'  
- Access frontend on localhost:3000.  
- Access backend on localhost:5000 or localhost:5000/tasks.  
- Use the app.  

***Approach 2 (Docker):*** Steps to run the Full-Stack app on Docker Containers:

Download Docker: Go to https://hub.docker.com, and choose the one for your OS.

From the dockerfiles in the repo, we will create two images and then two containers, one for Flask and one for React. To make these containers interact with each other we need to put them on the same network (since we are no longer working on the same local machine).  
- docker network create <network_name>

Backend: We need to first build an image. Then run container from that image.  
- On one terminal, navigate to the folder named backend where the dockerfile for Flask is.  
- Execute the command 'docker build -t <image_name> .'  
- Execute the command 'docker run --name <container_name> --network <network_name> -p 5000:5000 <image_name>  
Note: Port 5000 is where your backend is exposed. If you're new to this stuff, then you probably wanna look up small things like what 5000:5000 means.


Frontend: We need to first build an image. Then run container from that image.  
- On one terminal, navigate to the folder named frontend where the dockerfile for React is.  
- Execute the command 'docker build -t <image_name> .'  
- Execute the command 'docker run --name <container_name> --network <network_name> -p 3000:3000 <image_name>

Now that the app is running on the containers,
- Access frontend on localhost:3000.  
- Access backend on localhost:5000/ or localhost:5000/tasks or localhost:5000/users.  
- Use the app.

***Approach 3 (AWS Elastic Kubernetes Service):*** Steps to run the app on Kubernetes:

This is slightly complex when it comes to the set-up. It starts to make sense as you understand the architecture of what is happening. The point of using Kubernetes instead of just plain old dockers is that kubernetes helps with container orchestration: scaling, self healing, traffic load balancing etc. It is obviously not required for this project as it is a simple web app which I don't expect anyone to use but myself for learning purposes, but it is for these learning purposes that we implement kubernetes and figure out how the URLs change, how the builds change, and how to prepare your cluster to host your app. I also got a better understanding of dockers and AWS from learning the basics of kuberenetes. You can obviously use Chatgpt for the above or the below steps to learn better but I will give you my steps which are tailored to this app and you can execute faster (but at the cost of learning lesser). Learn among all the manifests, why we use deployments and services in the below steps.

- Set up an AWS EKS cluster using the official documentation (https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html, follow the guide thoroughly) (this will include prereq like aws cli etc)
- Create Service manifest - Type: Load Balancer for frontend and backend (or just use the ones I have written already). They will create load balancers which allow the traffic to be load balanced between the pod replicas. The purpose of this, or at least my understanding, is to give a common endpoint for the react app to be able to connect to the flask app. Since the containers are not in the same pods, they don't share a common namespace network and hence cannot reach each other through localhost. The nlb allows this common endpoint.
    - Deploy manifest: kubectl apply -f eks/frontend-service.yaml & kubecyl apply -f eks/backend-service.yaml
- Find out the DNS name for the backend loadbalancer, and replace all the localhost:5000 calls in the react code to the that DNS. For eg: loaclhost:5000/tasks becomes http://<lb-dns>/tasks
- Build react and flask images using docker build -t image_name:tag (start using tags because now you will see the need for them; replace tag with v1.0 etc)
- Since EKS will need to pull images from somewhere, let's push our local images built using dockers (previous step) to something like AWS ECR. This is done is four steps
    - login to AWS console and go to ECR. Create two repos and name them something like my-flask/react-ecr-image
    - aws ecr get-login-password --region <region_code> | docker login --username AWS --password-stdin <accnumber>.dkr.ecr.<region_name>.amazonaws.com      
    - docker tag <local_image_name:tag> <accnumber>.dkr.ecr.<region_code>.amazonaws.com/<repo_name:tag>
    - docker push <accnumber>.dkr.ecr.<region_code>.amazonaws.com/<repo_name:tag>
    - make sure the same is performed for the other frontend/backend too.
- Prepare Deployment manifest: The files are available under the eks folder for your reference. Change the ECR image URI to yours. Deploy both.
- Ensure everything is running: Do: kubectl get pods and kubectl get svc to make sure everything is running right. Make sure you have given the load balancer a few minutes to set up and the ECR repos are having your images in them.
- You should be able to visit frontend and backend on the loadbalancers' DNS.
___
**Tech Stack**

***React and Flask***

- React: Wrote this with ChatGPT as coding buddy. Two pages on this web app: SignIn/Register and Home Page to manage tasks.
    - Have used fetch to interact with flask instead of a library like axios.
    - Used JWT to manage session, upon logging in. Once logged out, session terminated. Till this was done, I was not able to stay logged in after refreshing.
    - Code isn't really documented, that is still pending. Feel free to do it and create a pull request.
    - App.js seems lengthy but it is a lot of logging and error handling and less logic, easy to understand. 
    - Created a flask_url variable to store which url to access flask app on. This will change depending on what the server is running on.
- Flask: Wrote this with ChatGPT too as coding buddy. Initially it was a simple one file code but later, to follow best practice, I split it into multiple packages.
    - Using REST API with user_routes and task_routes blueprinted into the main app.py.
    - Used CORS to allow access from React.
    - To refer to what the flask file initially looked like , back when there was no login page, you can open file app-archive. py in the backend directory.
    - You can use the above point to understand flask better and then dive into what it looks like with the current structure.

***CICD Using Gitlab-CI***

Note: A gitlab-ci file runs in stages. Each stage must have one or more jobs under it.
Two stages: Build (Two jobs: build_backend, build_frontend) and Deploy (deploys network and buils containers). Third stage Test yet to be included. 

Build job(s): moves into appropriate directories and builds images using the dockerfiles stored in those directories. Proceeds to compress and save the same in the form of artifacts. 

Deploy job: Creates a common network for both containers to connec to. Moves into appropriate directories and runs the docker containers on the loaded artifacts.  

___

***Terraform***

Infrastructre as code set-up for AWS resources. It is all documented in the terraform code, but you can find it here too.
- 2 ECR repos to store docker images and for EKS to pull images from.
- A fairly complex EKS set-up, following the official AWS guide.
    - Create CloudFormation stack provided by AWS for EKS (contents are in a file in the terraform directory)
    - Create IAM cluster role
    - Create policy attachment to cluster role
    - Cluster creation using outputs from network stack created by cloudformation above
    - Create IAM node role
    - Attach 3 policies to IAM node role
    - Create note group
- variables.tf for storing variables which can be accessed from other tf files.
- outputs.tf to store outputs created during resource creation.
- backend.tf has the tfstate file which has the current configurations, launched by terraform, on AWS stored. This allows us to have version controlled configurations.

___

**Major Learnings Per Stage**

I went into this project with very little knowledge on each stage. That was the whole point of doing the project, so I could learn! To an experienced engineer, this should not take much time but it took me a lot of time to overcome some fundamental issues. I would suggest you skip reading this part as the best way to learn is to bump into these issues yourself and grow from them. Come and refer to these as the last resort, when even the rest of the internet fails you! Below are my learnings.

***Frontend React:*** 
- The basic layout + syntax of React. Learnt how to use developer tools/web inspector to read http requests and responses. Learnt the importance of logging at different steps.
- How to use token storage for staying on a page post refresh and useNavigate to move around.

***Backend Flask:*** 

- The basic layout and syntax for Flask. Also, very important to add try and except blocks as they make error handling much easier and also give you a sense of what is going wrong.
- Learnt the need for CORS, thereby allowing frontend and backend to interact with each other. Only with flask, did my data on frontend persist.
- Learnt how to make the code work when the single flask file is split into multiple files and packages.
- Handling and returning JWT and data in general. I realized I was just returning message in json response. Later changed it to data being returned.
- How to route things for REST API.

***Docker*** 

- Since we are no longer deploying on our local, the way the frontend accesses the backend changes. 
- Till now flask was receiving requests from react on the same host. Now we are running flask on one container and react on another. So we need to first create a common network, so that the containers are running on the same network. 
- Even the flask code needs a minor change. The last line of code in flask is app.run(debug=True) when running the fullstack from the same host, but when it needs be accessed from another host on the same network, we need to change the line to app.run(host='0.0.0.0',debug='True')
- Everytime you install stuff for your code, update your requirements.txt so that the image builds correctly.
- when creating a dockerfile, create the directories properly so that the two programs can interact with each other through corrcet pathing.

***Kubernetes***

- react can no longer connect to flask over localhost:5000 but needs to use lb-dns (hence we deployed a load balacner type service). 
- add http:// now to the above or else it uses relative pathing and will end up combining two domain names of react+flask.
- dockers build can cache to build subsequrent build faster. This can lead to code updates not reflecting in the new builds.
- start using image:tags to keep track of which version you are on.
- keep an eye on which instances are deployed by the node group. My t3.mediums started costing me.
- Don't need more than 1 replica for this project so make it 1 in the deployment manifest.

***AWS & Terraform***

- AWS: While running my cluster, as expected, I had incurred a small expnse of $3. When I checked my billings page, I could see it was from EC2 and the EKS cluster. I went ahead and deleted the two after I was done deploying my app succesfully. However, when I logged in after 10 days, I saw that I was charged 2$ everyday and the reports page was still saying EC2, but I had no instances running. I then realized that it must be the Elastic IPs, security groups, and other networking resources which were created by the CloudFormation template. Hence, delete cluster, roles & policies, and then also cloudformation stack. If you use terraform for the above, then terraform destroy should take care of everything. One of the biggest advantages of using Terraform. Keep visitng the bill and cost explorer page on AWS to keep a track of expenses.
- Terraform: use depends-on to reduce errors when applying.
- Terraform: Don't forget to destroy after testing apply.
- Terraform: Set instance type to t3.micro to prevent node group from deploying other more expensive instances.

***Unit Tests***

- Run pytest from the backend directory and not from the test directory. There is a particular way of naming test files because pytest looks for that.
- Need to run all db operations withing 'with app.app_context():' block or else it will fail.
- For npm tests for react, the dependencies shouldn't have different version of jest installed. That is what has currently halted my progress in writing tests.

***CICD Pipeline***

- Need to call a base docker image (dind) or else your docker commands will fail.
- Each job runs on a separate environment/host, hence it adds complexities of passing the built image/network into the next jobs.
- Each job starts from the base directory.
- the '.' at the end of the docker build indicates to build it in the current directory, however this was not a requirement when I was doing it locally (maybe different default settings for not mentioning which directory).
- When you want to save something and upload it as an artifact, the path you give is where it searches for that particular document. Then when you load this saved document, it also searches in that same path.
- You cannot deploy frontend and backend in two different jobs because they have to connec to same network, and that network has to be in the same job. 
- It is very difficult to access the web app from the deployed container.
___
**Issues:** Can solve these with some effort but prioritizing finishing a skeletal structure for now.

- Issue 1: Having to refresh the web page to see the added tasks.  
- Issue 2: Having to refresh the web page to see deleted tasks, or else it is going from 'Done' to 'To Do'. Mostly linked to the above issue.
- Issue 3: Accessing the web app after launching it on the gitlab runner.
- Issue 4: Should the containers be deployed on to an EC2 instance? Easier to access the web app on EC2.
- Issue 5: Including tests in CICD pipeline.

Please feel free to reach out if you want to pursue this project and have any doubts. Guiding others will only improve my knowledge. Wishing you the best!


