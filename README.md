# A CI/CD pipeline project for a Trunk-Based Development strategy in a Kubernetes environment

   The main purpose of this project is to describe and deploy a pipeline for Continuous Integraton and Continuous Deployment of a Trunk-Based Development Strategy in a Kubernetes environment, leveraging opensource tools in order to foster automation, control and agility.

# The Trunk-Based Development (TBD) Strategy
  Many organizations are now working with (or migrating to) TBD. 
# CI/CD - continuous integration and deployment pipeline topology

The following pipeline was designed to enable CI/CD in a kubernetes cluster, using:

- Argo-cd - https://argoproj.github.io/argo-cd/ - Continuous deployment tool and health monitoring on staging and prod
- Drone.io - https://drone.io/ - pipeline automation
- Harbor - https://goharbor.io/ - Images and Helm Chart Repository
- gitea - https://gitea.io/ - git based service for code hosting
- helm - https://helm.sh/ - tamplating and packaging for kubernetes objects
- kubernetes - https://kubernetes.io - Container orchestration infrastructure

The designed and implemented topology is:

![Alt text](https://github.com/alexismaior/cicd/blob/master/pipeline.png?raw=true "Title")

