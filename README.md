# OpenShift Application Developer Training

This workshop is designed to take a developer on a journey through Cloud Native application development on OpenShift. The workshop looks at how a developer can begin with using container applications running on a local container runtime such as Docker or Podman and expanding the core concepts to create OpenShift applications through a variety of deployment methods, ranging from simple YAML definitions to CICD pipeline deployments.

# Agenda

## Day 1
**Cloud native application architecture**
- Cloud–native overview
- Introduction to Containers
    - Overview of container technology
    - Container architecture
	- Containers vs traditional applications
	- Deploying containers
	    - Deploying stateless containers
		-  Deploying stateful containers
	- Container images and repositories
	- [DEMO] Deploying a container
	- [DEMO] Creating a Docker image and deploying a container
	- [LAB] Deploying a stateless container
	- [LAB] Deploying a stateful container
	- [LAB] Creating custom container applications
- Introduction to Red Hat OpenShift
	- OpenShift/Kubernetes Core Concepts
	- OpenShift Components
	- [DEMO] OpenShift User Interface 
		- Admin interface
		- Developer interface
    - Exploring OpenShift
- OpenShift 4 Architecture
- OpenShift Container Security
    - Security Considerations
- Security Context Constraints
- OpenShift Container Networking
	- Container to container networking
	- OpenShift Routes
- Introduction to OpenShift Storage
	- Persistent Storage Overview
	- User provisioned vs Dynamic Storage


## Day 2
**Develop and deploy apps on OpenShift**
- Introduction to OpenShift Application Resources
	- Application High Availability with Deployments
	- Application Configurations with ConfigMaps
	- Application sensitive data with Secrets
	- Exposing the application with services and routes
- OpenShift Application Deployments
	- Deploying applications on OpenShift
	- [DEMO] Creating an OpenShift application
	- [LAB] Creating an OpenShift application
	- [LAB] Advanced OpenShift application - Wordpress + MySQL
    - OpenShift Storage for Applications
        - Persistent Storage Recap
    	- [LAB] Creating a database with user provisioned storage
	    - [LAB] Creating a database with dynamic storage
	- Deploying applications using the user interface
	- [DEMO] Deploying applications using the user interface
	- [LAB] Deploying applications using the user interface
    - Source2Image application deployments
	- [DEMO] S2I deployments
    - [LAB] S2I deployments
    - Operators
    - [LAB] Operator deployment


## Day 3
**Develop and deploy apps on OpenShift continued…**
- Introduction to Helm charts
	- Helm 3 overview
	- Developing helm charts
	- Deploying Helm charts to OpenShift
    - [DEMO] Developing, deploying and updating a simple Helm 3 chart
	- [LAB] Developing and deploying a simple Helm 3 chart
- OpenShift Security Context Constraints
	- Restricting workloads permissions with SCCs
	- [DEMO] Restricting workloads permissions with SCCs
- Limiting resources with Quotas
	- Restricting workload resource usage with quotas
	- [DEMO] Restricting workload resource usage with quotas
- [LAB] Using SCCs and Quotas to restrict applications
- Image Registries
    - OpenShift Image Registry
    - [DEMO] Pushing and pulling images
    - [LAB] Pushing and pulling images
- Scaling Applications
    - Manual and Autoscaling Applications
    - [LAB] Scaling applications


## Day 4
**DevOps: continuous delivery**
- DevOps and DevSecOps
- WebSphere Liberty on OpenShift
- [LAB]: Set up a CI/CD pipeline on OpenShift using Jenkins to deploy a simple web application
- Transformation Advisor - Migrating WebSphere Applications to OpenShift

**Microservices architecture**
- Microservices application architecture
- Developing microservices
- Twelve factor applications
- Refactoring monolith applications into microservices
- [LAB] Build and deploy a polyglot microservices application on OpenShift

## Day 5
**Microservices architecture**
- Red Hat OpenShift ServiceMesh
- Microservices security
- [LAB]: Using ServiceMesh to deploy a microservices application