# springboot-k8s-yaml
1) Create a spring boot project
2) Create a docker file
3) Create a local image of Docker
4) Push image to docker hub
5) Pull image from docker hub
6) Run project using image of docker hub

docker build -t springboot-k8s-demo .	#Create image in local <br>
docker tag 0.0.1-SNAPSHOT pabitramaiti/springboot-k8s-demo	#Create tag of image <br>
docker push pabitramaiti/springboot-k8s-demo <br>
-- docker pull pabitramaiti/springboot-k8s-demo:0.0.1-SNAPSHOT <br>
docker run -p 8080:8080 pabitramaiti/springboot-k8s-demo:latest		# it is first download image from a registry (such as Docker Hub or a private registry) and stores it locally on your machine then create and start a new container from an existing image.

Or Directly push without running tag command:-
==============================================
docker build -t pabitramaiti/springboot-k8s-demo:0.0.1-SNAPSHOT .
docker push pabitramaiti/springboot-k8s-demo:0.0.1-SNAPSHOT
-- docker pull pabitramaiti/springboot-k8s-demo:0.0.1-SNAPSHOT
docker run -p 8080:8080 springboot-k8s-demo:0.0.1-SNAPSHOT 	# it is first pull image from docker hub then run the application

docker pull:-
===========

The docker pull command is used to download a Docker image from a registry (such as Docker Hub or a private registry).
It fetches the specified image from the registry and stores it locally on your machine.
It does not create or start a container; it only retrieves the image.

docker run:-
===========
The docker run command is used to create and start a new container from an existing image.
If the specified image is not already available locally, docker run will automatically call docker pull to download the image.
It creates a new container instance based on the image and runs the specified command inside that container.

Setup Kubernetes in Windows & Run Spring boot application on k8s cluster 
========================================================================

https://www.youtube.com/watch?v=xhxmExC9N1U&list=PLVz2XdJiJQxybsyOxK7WFtteH42ayn5i9&index=4
https://www.youtube.com/watch?v=xhxmExC9N1U&list=PLVz2XdJiJQxybsyOxK7WFtteH42ayn5i9&index=5
https://medium.com/@javatechie/kubernetes-tutorial-setup-kubernetes-in-windows-run-spring-boot-application-on-k8s-cluster-c6cab8f7de5a

docker context use default
minikube start --driver=docker
minikube status
kubectl cluster-info
kubectl get node
minikube docker-info
## To point your shell to minikube's docker-daemon, run:
minikube -p minikube docker-env --shell powershell | Invoke-Expression   # This command allow kubernetes to read your docker repository.
docker images
kubectl create deployment spring-boot-k8s --image=springboot-k8s-demo:0.0.1-SNAPSHOT --port=8080   # This is similar to "kubectl apply -f deployment.yaml", here only without deployment.yaml file. Mannually create deployment object.
kubectl get deployments
kubectl get deployments spring-boot-k8s  # Describe summary of deployment
kubectl describe deployment spring-boot-k8s  # Describe details of deployment
kubectl get pods
kubectl expose deployment spring-boot-k8s --type=NodePort   # Command to create service obsject. This is similar to "kubectl apply -f service.yaml", here only without service.yaml file. Mannually create service obsject.
kubectl get service
minikube service springboot-k8s-svc --url	#Start tunnel or get the proxy url of service to access it .
http://127.0.0.1:53509	# Service Endpoing: http://127.0.0.1:53509/message
kubectl delete service springboot-k8s-svc	# Delete Service
kubectl delete deployment spring-boot-k8s	# Delete Deployment
minikube stop	# Stop minikube
minikube delete		# minikube delete


Deploying Spring Boot on AWS EKS (Elastic Kubernetes Service )
================================================================

https://www.youtube.com/watch?v=mVSFHgItaa4&list=PLVz2XdJiJQxybsyOxK7WFtteH42ayn5i9&index=11
https://medium.com/@javatechie/deploying-a-spring-boot-application-on-aws-eks-fdd7d075f034

Before deploying Spring Boot application in EKS cluster you need to install AWS CLI and eksctl in your system. But it is not mandatory to install both, yu can configure same in AWS console it self.
https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/setting-up-cli.html    --> Install the AWS CLI
https://chocolatey.org/install --> Install Chocolatey first then instal eksctl
https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/setting-up-eksctl.html --> Install eksctl

Open command prompt

D:\Core_Services\springboot-k8s-demo> aws configure
AWS Access Key ID [None]: AKIA3N55WRHMA64FCAOG
AWS Secret Access Key [None]: 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
Default region name [None]: ap-south-1
Default output format [None]:

Elastic Container Registry (ECR) Setup
1) Go to Amazon ECR.
2) Create a repository -> springboot-k8s-demo
3) Click on newly created repository in ECR service.
4) Click on "View Push Commands"
5) Open command prompt and execute below script step by step.

> aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 785839983064.dkr.ecr.ap-south-1.amazonaws.com
> docker build -t springboot-k8s-demo .
> docker tag springboot-k8s-demo:latest 785839983064.dkr.ecr.ap-south-1.amazonaws.com/springboot-k8s-demo:latest
> docker push 785839983064.dkr.ecr.ap-south-1.amazonaws.com/springboot-k8s-demo:latest
OR//
# docker build -t 785839983064.dkr.ecr.ap-south-1.amazonaws.com/springboot-k8s-demo:0.0.1-SNAPSHOT .
# docker push 785839983064.dkr.ecr.ap-south-1.amazonaws.com/springboot-k8s-demo:0.0.1-SNAPSHOT

6) After finising the push image to ECR repository, Click ‘COPY URI’ to copy the image URL and save it for use in the upcoming steps. [ 785839983064.dkr.ecr.ap-south-1.amazonaws.com/springboot-k8s-demo:latest ]
7) Setup EKS cluster. There are two ways to create EKS Cluster. (1) Using eksctl (2) AWS Management Console.
8) Lets see how we can setup cluster using eksctl command.
9) Create EKS cluster (Run below commands)

eksctl create cluster --name javatechie-cluster --version 1.29 --nodes=1 --node-type=t2.small --region ap-south-1

# eksctl delete cluster javatechie-cluster
# eksctl delete cluster --name my-cluster --region region-code
aws eks --region ap-south-1 update-kubeconfig --name javatechie-cluster
kubectl apply -f k8s_for_eks.yaml
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl get service

D:\Core_Services\springboot-k8s-demo>kubectl get service
NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP                                                               PORT(S)        AGE
kubernetes           ClusterIP      10.100.0.1      <none>                                                                    443/TCP        19h
springboot-k8s-svc   LoadBalancer   10.100.72.249   a8faf61817cab48e0a6a8bdfa1ba1b54-325727299.ap-south-1.elb.amazonaws.com   80:31783/TCP   9m17s

Service endpoint:- http://a8faf61817cab48e0a6a8bdfa1ba1b54-325727299.ap-south-1.elb.amazonaws.com/message

kubectl get service / svc
kubectl delete service springboot-k8s-svc
kubectl get deployment
kubectl delete deployment spring-boot-k8s
eksctl delete cluster javatechie-cluster  # Tear down the cluster


