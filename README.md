# A CI/CD pipeline project for a Trunk-Based Development strategy in a Kubernetes environment

   The main purpose of this project is to describe and deploy a pipeline for Continuous Integraton and Continuous Deployment of a Trunk-Based Development Strategy in a Kubernetes environment, leveraging opensource tools in order to foster automation, control and agility.

# The Trunk-Based Development (TBD) Strategy
  Many organizations are now working with (or migrating to) TBD. As from https://trunkbaseddevelopment.com/, this model aims to reduce the effort on merging procedures, focusing on maintaining a constant buildable "master" branch. Because of its simplicity, it integrates seamlessly with CI/CD.

  In this project, it has been considered, along with the master branch, feature branches, which can be short or long-lived. It is up to the developer to decide, provided that the long the life of a feature-branch, the more challenging the merge process will be.

  In my organization, every commit (and push) to master branch is considered a "good to go" build. The validation process with the client is done either in the feature branch, before the merge with master, or in production, leveraging toggle features, which can be enabled only in staging environment. The devops lemma in our team is: "The faster we fail, the faster we fix it".

  The basic process below outlines the actual TBD pipeline:
  ![Alt text](https://github.com/alexismaior/cicd/blob/master/tbd-process-v1.png?raw=true "TBD")

  It's in discussion an improvement of this process, where the Release Manager role will be responsible for tagging the release whenever it is ready for staging and production stages. This will move the "authorization" fase one step earlier. It is also under the radar automation between the dev Workflow Management tool and git tool, enabling auto tagging when the ticket (or user story) was validated by the release manager.

  In this scenario, the TBD pipeline would be something like this:
  ![Alt text](https://github.com/alexismaior/cicd/blob/master/tbd-process-v2.png?raw=true "TBD new")

  For this project, the first TBD will be considered for CICD implementation.

# The Tools

In devops world, it can be extremely challenging to find the perfect tool. There are hundreds available which offers amazing features. There are, however, some differences that can make a them particularly interesting for a project. At the end of the day, it is a matter of taste, usability, features and integration to your environment. In our case, adding to those, being an open source tool was also a requirement. A good way to start is to check if the project is supported by the CNCF (Cloud Native Computing Foundation - https://landscape.cncf.io/). As we have already decided for orchestrating containers with kubernetes, having a k8s native tool was also a plus. Furthermore, all tools, applications and systems were deployed in a fully managed private cloud. This is definitely not a constraint. Rather, being a cloud-ready project, all infrastructure can be easily migrated to a hybrid or public cloud provider, such as aws, azure or gcp.  

This is our chosen fleet:

- Argo-cd - https://argoproj.github.io/argo-cd/ - "Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes". We decided to use Argo-cd as the deployment tool for deploying helm charts in staging and production environments. It provides not only visibility and rollbacks, but also the option to manually sync new available helm charts to prod. This complies with the authorization requirement for publishing in production. Those who have proper rights can sync. In staging, nevertheless, the synchronization is automatic. Whenever a new helm chart for that project became available on the chart repository, Argo-cd takes care of its deployment. This means that it also enforce configuration. If someone accidentally change or deletes an object in the k8s namespace watched by argo triggers auto-sync (or not-in-sync in prod). Argo-cd has recently joined CNCF as incubating project.
- Drone.io - https://drone.io/ - Pipeline automation. This is the core tool that enables de CICD pipeline. Drone.io is an amazing project that has been recently acquired by Harness.io. The pipeline is described within the builded project in a drone.yaml file. It is a docker based pipeline tool. This means that every step in the pipeline is a container, that dramatically enhanced performance with reduced resource consumption. Describing the whole pipeline in a drone.yaml file in each repository can be a problem when you have an application with dozen or even hundreds of microservices managed by independent repos. Luckly, drone.io provides the possibility to centralize configuration in only one repo and describe the rules for forwarding the proper drone.yaml by configuring a drone-config service (https://github.com/drone/boilr-config). Drone is a stable tool that obviously has a lot to evolve, but is a promising project that will definitely worth a look.
- Jenkins - https://www.jenkins.io/ - The well-known alternative for drone.io. As drone still don't support pipeline triggers based on branch deletion (as its inner logic is based on a drone.yaml file available on that branch), we decided to use jenkins only to execute the deletion of a "feature" k8s cluster every time a feature branch is deleted.
- Ansible - https://ansible.com. Automation and configuration management of infrastructure and applications. Ansible was used in this project to automate the creation and deletion of a k8s cluster to host a feature-branch build.
- Harbor - https://goharbor.io/ - Docker Images and Helm Chart Repository. It has also native integration with Clair and Trivy image security scanners. It is possible to define security tresholds for pulling images. If an image is tainted with critical vulnerability, the pull for that image will fail, making possible to use this feature in the Security Test step.
- Gitea - https://gitea.io/ - git based service for code hosting. It is a great option for self-hosting a git service on premise.
- helm - https://helm.sh/ - tamplating and packaging for kubernetes objects. With helm, it is possible to manage the creation and deployment of all kubernetes declarative files (deployment.yaml, service.yaml, ingress.yaml, etc) by templating them. In this project, we defined only one template for all repos. This has enabled centralized configuration with simplified management. The trade off are more complex templates.
- kubernetes - https://kubernetes.io - Container orchestration infrastructure. The foundation platform for our container-based projects.
- VMWare - https://vmware.com - Private cloud infrastructure.

The designed and implemented topology is:

![Alt text](https://github.com/alexismaior/cicd/blob/master/pipeline.png?raw=true "Title")
