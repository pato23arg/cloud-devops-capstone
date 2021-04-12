# Cloud DevOps Engineer Capstone Project

 Final project of the DevOps Udacity Nanodegree

## Introduction

In this project I will apply the skills and knowledge which were developed throughout the Cloud DevOps Nanodegree program. These include:

- Working in AWS
- Using CircleCI to implement Continuous Integration and Continuous Deployment
- Building pipelines
- Using EKS CircleCI Orb to deploy clusters
- Building Kubernetes clusters
- Building Docker containers in pipelines

## Application

In this capstone, we will reuse the Python app used in project #4 to that implements ML to make predictions, and publish a Flask API:

```
kubernetes manifest files in /kubernetes folder. 
```

Application brief explanation.

## Kubernetes Cluster

The EKS Orb is used to deploy the Kubernetes Cluster. The code is in `cloudformation` directory. It is divided in two stacks, one for the deployment (kube-deployment-blue/green.yml) and a second one for the loadbalancer service (service-green/blue.yml).

This will create the stack in the following order:

1. VPC in eu-east-1.
2. Internet Gateway attached to the VPC.
3. Two public subnets in different AZs.
4. Public route table associated with the two public subnets.
5. Security group for the cluster.
6. Kubernetes cluster.
7. Kubernetes node group.


## CircleCI Pipeline

I am using a CI/CD pipeline on a blue/green deployment. On each new <<app>> generated, the docker image is updated, pushed to DockerHub and the Kubernetes pods are deployed so they start with the new image from the repository. There is a load balancer before the exposed 80 TCP ports to avoid having downtime. These are the steps in more detail:

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

This is the output when LINT fails:

![FAIL Lint](./screenshoots/LINT_fail.JPG)

And this is the output when LINT passes:

![SUCCESS Lint](./screenshoots/LINT_success.JPG)
  
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
