# Project 6: Containerising a webapp
In Projects 1 & 2, our webapp was locally set up step by step on a local machine. This was very necessary for the present project since to containerize a webapp, a prior mastery of the different setup steps is required. In this project, the configuration steps were rather containerized using Docker. I used base images(openjdk, tomcat, nginx) to configure and build the images for each tier(three tiers) to work according to our architecture. To reduce the image size, I used a multistage dockerfile to build and run our artifact, since building artifact needed so many downloaded dependencies. In order to build and test the different images simultaneously, docker-compose was very helpful. Containerizing microservices solves the problems of resource wastage, human errors during deployments. It just deploys on as much resources as needed.

## Project architecture
![]()
