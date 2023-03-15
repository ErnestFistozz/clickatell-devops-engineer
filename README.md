# Design Overview
![design](.\clickatell_design.png)
# Generating Spring Boot project
The first step is generating the spring boot application with the spring initialiser found [here](https://start.spring.io/) . A sample used can be seen below:
![sample](.\spring_initializer.png)

## Application Design

### Architecture
Model
| name | defition |
--------|----------|
`firstName` | first Name of engineer |
`lastName` | surname of engineer |

### Dockerization of application
The Dockerization for the application is dependend on the version of java used. Here is a sample of what I would do. The base image would be from `openjdk`.  Secondly, i prefer running containers as non-root, as such i would create a user (in this sample i am calling it devops and  the group the user belongs to is called devops as well). After which I create directory inside the container which will be the app directory and give my group and user permission to read and execute

> *Interestingly this is an official image, yet laballed as `pre-releases` and `non-production` images*
```json
FROM openjdk:17-oracle
LABEL description="DevOps Engineer API"
USER root
RUN mkdir -p /usr/app

ARG USER_ID=1000
ARG GROUP_ID=1002
ARG GROUP_NAME=devops

RUN group -g ${GROUP_ID} ${GROUP_NAME} &&\
        useradd -l -u ${USER_ID} -g ${GROUP_NAME} ${GROUP_NAME} &&\
        install -d -m 0755 -o ${GROUP_NAME} -g ${GROUP_NAME} /usr/app

COPY target/DevopsEngineer-0.0.1-SNAPSHOT.jar /usr/app
EXPOSE 8000
USER ${GROUP_NAME}
ENTRYPOINT [ "java", "-jar", "DevopsEngineer-0.0.1-SNAPSHOT.jar"]
```
### Publishing image
I would create a registry in azure and publish the image as part of the build pipeline:
Each build build will publish the image as

```bash
$ docker build --file $(Build.SourcesDirectory)/Dockerfile -t devops-engineer:v1.0.0-$(Build.BuildId)
$ docker tag devops-engineer:v1.0.0-$(Build.BuildId) <registry-name>.azurecr.io/repo/devops-engineer:v1.0.0-$(Build.BuildId)
$ docker push registry-name>.azurecr.io/repo/devops-engineer:v1.0.0-$(Build.BuildId)
```

> **NB:** *`I would preferrably download and store the openjdk image from dockerhub and store it in the azure registry to have a centralised placed`* <br>
*`before the image can be pushed or downloaded from the azure registry, a form of authentication and authorization is required via the <docker login> command`*

# CI/Build Pipeline
-  I would choose the `Azure DevOps` service from azure. First its free and very easy to use. It offers both YAML based and UI (classic) pipelines. Moreover, Microsoft offers hosted agents (build servers - containers to be exact) with which you can use to do your build perspective. This reduces the maintenance overhead from a agent management perspective. Microsft agents come pre-installed with pretty much "all" build tools required in most cases, and allows for run time installation of tools and their corresponding versions.

> **It is worth noting that this would be ideal for build pipelines that do not interact with resources that are hosted in a private network** 

# Release/Deployment Pipeline


# Infrastructure Provisioning

AKS infrastructure would be used as the hosting platform for the application. The provisioning tool would be terraform for consistency and ease of provision and infrastructure state managent.
> *I won't deal with the actual provisioning of the infrastructure here*

## Requirements to provision AKS

- `Subscription` --> billing mechanism
- `Resource Group` --> allows for the logical group of resource
- `Virtual Network` and `Subnet` --> private network address and subnet address space
- `Service Principal` -- corresponds to the app registration APP Id and its corresponding Secret. This is the `user` we use to deploy the infrastructure. The app registration and role assingments need to done before provisioning the infrastructure
- `Key Vault` --> stores credentials including the service principal creds
- `log analaytics workspace` - used for logging and monitoring AKS 
- `storage account` (general purpose v2 - cool tier) --> and container  are required to store the terraform state file

# Ingress and loadbalancer

# helm


1. Locally, `kubectl` tool is required if you wish to interact with the cluster