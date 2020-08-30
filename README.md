# A CI/CD pipeline project for a Trunk-Based Development strategy in a Kubernetes environment

   The main purpose of this project is to describe and deploy a pipeline for Continuous Integraton and Continuous Deployment of a Trunk-Based Development Strategy in a Kubernetes environment, leveraging opensource tools in order to foster automation, control and agility.

# The Trunk-Based Development (TBD) Strategy
  Many organizations are now working with (or migrating to) TBD. As from https://trunkbaseddevelopment.com/, this model aims to reduce the effort on merging procedures, focusing on maintaining a constant buildable "master" branch. Because of its simplicity, it integrates seamlessly with CI/CD.

  In this project, it has been considered, along with the master branch, feature branches, which can be short or long-lived. It is up to the developer to decide, provided that the long the life of a feature-branch, the more challenging the merge process will be.

  In my organization, every commit (and push) to master branch is considered a "good to go" build. The validation process with the client is done either in the feature branch, before the merge with master, or in production, leveraging toggle features, which can be enabled only in staging environment. The devops lemma in our team is: "The faster we fail, the faster we fix it".

  basic process of TBD process and pipeline is:
  ![Alt text](https://github.com/alexismaior/cicd/blob/master/tbd-process?raw=true "TBD")

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
