# Who Runn The World app? CATs or DOGs ?

A simple distributed application running across multiple Docker containers.

## Requirement

- The project need to be done inside a virtual machine that run docker and docker compose. Follow the docker documentation as we saw during the course for installation instructions if you don't have it yet.

## Getting started

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

## Architecture

![Architecture diagram](architecture.png)

- A front-end web app in [Python](/vote) which lets you vote between two options
- A [Redis](https://hub.docker.com/_/redis/) which collects new votes
- A [.NET](/worker/) worker which consumes votes and stores them in…
- A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
- A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The `WhoRunTheWorld` application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.

# Requirements

This project involve deploying a kubernetes cluster and the VMs that you use **MUST** be public vms. Local vms will not allow you to achieve all the tasks.
You MUST create 3 virtual machines. One named `master` and the other ones `worker1` and `worker2`

# Taks

## Containers orchestration / Kubernetes

1. Create an image for each service. I made things easy for you by providing the `Dockerfile` that is needed for each service. ENJOY ! :)
2. Create a `deployment` for each service (`database`, `redis`, `result`, `vote`, `worker`)
3. Create a cluster IP service vor services that do not need public exposure
4. Create a NodePort service for service that require public exposure
5. Make sure you use `ConfigMap` for everything related to application configs and `Secrets` for everything related to sensitive data.

   - Please NEVER EVER commit sensitive informations in cloud. You can replace the values of sensitive informations with <PLACE_HOLDERS>

**_Note:_** Please be aware that I have provided a docker compose yaml file that you can refer for some useful informations in order to build your kubernetes manifests.

## CICD

1. Create a CICD pipeline that automatique the deployment of the whole app (all the services)
2. Make use of Kubernetes namespaces in order to have `dev`and `prod` environments.
3. Please make sure your pipeline deploy different version of the app in each environement.

## BONUS

Create a terraform deployment that automatically provision the virtual machines

# Submission

- You need to submit your own public github repository with the required files that demonstrate your work.
- You need to write a complete, clean README like a pdf rendering (summary, table of content, sections, images etc...)
  The README should contain everything needed to demonstrate your work
- You need to write a clean pdf documente that present your work in details
- For the README file, it should be called `SUBMISSION.md`
- The PEPAL submition named `WhoRunTheWorld` is created for you to submit your work.

# You need to submit before 17AM today.
