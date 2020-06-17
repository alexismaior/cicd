# CI/CD - continuous integration and deployment pipeline topology

The following pipeline was designed to enable CI/CD in a kubernetes cluster, using:

- Argo-cd - https://argoproj.github.io/argo-cd/ - Continuous deployment tool and health monitoring on staging and prod
- Drone.io - https://drone.io/ - pipeline automation
- Harbor - https://goharbor.io/ - Images and Helm Chart Repository
- gitea - https://gitea.io/ - git based service for code hosting
- helm - https://helm.sh/ - tamplating and packaging for kubernetes objects
- kubernetes - https://kubernetes.io - Container orchestration infrastructure

The designed and implemented topology is:

