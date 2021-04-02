# Cloud DevOps Engineer Capstone Project

 Final project of the DevOps Udacity Nanodegree

## Introduction

In this project I will apply the skills and knowledge which were developed throughout the Cloud DevOps Nanodegree program. These include:

- Working in AWS
- Using CircleCI to implement Continuous Integration and Continuous Deployment
- Building pipelines
- Using Ansible and CloudFormation to deploy clusters
- Building Kubernetes clusters
- Building Docker containers in pipelines

## Application

Explain Wordpress or Hipster Shop app (need to choose wich one to use):

```
kubernetes manifest here 
```

Application brief explanation.

## Kubernetes Cluster

Cloudformation is used to deploy the Kubernetes Cluster. The code is in `cloudformation` directory. It is divided in two stacks, one for the network (kube-network.yml) and a second one for the cluster itself (kube-cluster.yml).

There are scripts to create and update the stacks. For example, to create both stacks:

```
./stack-create.sh udacity-net kube-network.yml kube-network-parameters.json
./stack-create.sh udacity-eks kube-cluster.yml kube-cluster-parameters.json
```
This will create the stack in the following order:

1. VPC in eu-east-1.
2. Internet Gateway attached to the VPC.
3. Two public subnets in different AZs.
4. Public route table associated with the two public subnets.
5. Security group for the cluster.
6. Kubernetes cluster.
7. Kubernetes node group.

The node group is created in EC2:


## CircleCI Pipeline

I am using a CI/CD pipeline on a blue/green deployment. On each new <<app>> generated, the docker image is updated, pushed to AWS and the Kubernetes pods are rolled out so they restart with the new image from the repository. There is a load balancer before the exposed 80 TCP ports to avoid having downtime. These are the steps in more detail:

<<pic here>>

1. <<app>> Reqs.

The script's Python requirements are installed. They include <<reqs>>.

```
pip3 install -r requirements.txt
```

2. Python Lint

Check the script's code with pylint. 

```
pylint --disable=W0311 app.py
```

This is the output when the steps fails:

<<failed lint pic>>

And this is the output when it passes:

<<success lint pic>>
  
3. Run App

the webpage is operational

```
python3 app.py
```

4. Build & Push

The docker image is build using the Dockerfile and the image is pushed to Amazon Elastic Container Registry.

```
aws ecr get-login-password --region eu-east-1 | docker login --username <<user>> --password-stdin <<pass>>
docker -H=tcp://localhost:2375 build -t udacity-capstone .
docker -H=tcp://localhost:2375 tag udacity-capstone:latest <<docker eks registry>>
docker -H=tcp://localhost:2375 push <<docker eks registry>>
```

5. Docker Container

A Docker Container is started to test if there is any problem using the image.

```
docker -H=tcp://localhost:2375 container run <<docker eks registry>>
```

6. Deployment

The Kubernetes pods are deployed using the pushed image. This is a no-op if the deployment files have not changed.

```
kubectl apply -f deployment.yml
kubectl apply -f loadbalancer.yml
```

7. Rollout Deployment

Rollout the pods to use the new pushed image.

```
kubectl rollout restart deployment/webserver
```

## View the public web page

Finally, enter the public DNS record in the web browser to access the page.

```
$ kubectl get deployment
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
webserver   1/1     1            1           7m30s
$ kubectl get service
NAME           TYPE           CLUSTER-IP    EXTERNAL-IP                                                              PORT(S)        AGE
kubernetes     ClusterIP      172.20.0.1    <none>                                                                   443/TCP        19m
loadbalancer   LoadBalancer   172.20.75.5   <<service-url>>   80:31022/TCP   
```

<< app webpage >>
