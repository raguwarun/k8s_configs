# Setup Kubernetes (K8s) Cluster on AWS
Create Ubuntu EC2 instance

Install AWSCLI and pip/python
	sudo apt-get update -y
	sudo apt-get install awscli
   
Install kubectl
   curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
   chmod +x ./kubectl
   sudo mv ./kubectl /usr/local/bin/kubectl
You can also install from https://kubernetes.io/docs/tasks/tools/install-kubectl/

Create an IAM user/role with Route53, EC2, IAM and S3 full access
 	Attach IAM role to ubuntu server

Note: If you create IAM user with programmatic access then provide Access keys.

Install kops on ubuntu instance:
 	curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops- linux-amd64
 	chmod +x kops-linux-amd64
 	sudo mv kops-linux-amd64 /usr/local/bin/kops
You can also install from https://github.com/kubernetes/kops/blob/master/docs/development/building.md

Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)

	Eg: k8s.local 

Choose a cluster name
 	export NAME=ragu.k8s.local

Create an S3 bucket
 	aws configure
 	aws s3 mb s3://kubconfig.k8s.local
	
Expose environment variable
 	export KOPS_STATE_STORE=s3://kubconfig.k8s.local
	
Create sshkeys before creating cluster
 	ssh-keygen
	
Create kubernetes cluster definitions on S3 bucket
	kops create cluster --cloud=aws --zones=us-east-1a --name=ragu.k8s.local --dns-zone=k8s.local --dns private --authorization RBAC --master-size t2.micro --master-volume-size 8 --node-size t2.micro --node-volume-size 8 --yes

Validate your cluster
 	kops validate cluster --wait 10m
	
To list nodes
  	kubectl get nodes 
	
Deploying Nginx container on Kubernetes
	kubectl run sample-nginx --image=nginx --replicas=2 --port=80
   kubectl get pods
   kubectl get deployments
 
Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them:
	kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
 	kubectl get services -o wide
 
To delete cluster
	kops delete cluster ragu.k8s.local --yes

