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

# The pipeline

  So, let's put all together. Drone.io is responsible for the underlying pipeline automation. All steps in the pipeline are commanded or orchestrated by drone and its plugins.
  This project will not delve into configuration of the pipeline's components. All config is specific for the environment and must be adapted for each situation. In this project, all tool are already running in a kubernetes cluster (except jenkins, which was already installed in a standalone VM).

![Alt text](https://github.com/alexismaior/cicd/blob/master/pipeline-v0.png?raw=true "Pipeline")

  First, let's consider a commit in the master branch and a push to Gitea for a random repo.

- Gitea is configured to send webhooks to Drone to activate the pipeline process. This configuration is easy an can be done by creating an OAuth Applicaton in Gitea (https://docs.drone.io/server/provider/gitea/). By default, Gitea will send webhook events for branch or tag creation, pull requests and pushes. But this can be also configured.
- Push: Drone server will receive this event and will call drone-config service (another container from a boiler plate project) to retrieve the proper centralized drone.yaml file, which is also versioned in a git repo. It is possible to write a logic for this step, in case you have different pipelines for different repos.
- Drone server will then pass this drone.yaml to drone agent, the third "Drone suite" component. Drone agent is responsible to parse this yaml file and call the plugins. Plugins in drone are nothing more than docker containers running specific images to execute the step. There are many available in http://plugins.drone.io/. The first (and default) step for drone is to clone the target repo into a workspace. This workspace is a docker temporary volume that is mounted on each container for each step. This is quite useful to pass information for one step to another. Drone will then execute the following steps.
![Alt text](https://github.com/alexismaior/cicd/blob/master/drone-master-v1.png?raw=true "Drone-master")

- Unit Test: Drone will then execute the unit tests defined in the repo.
- Build and Push: Drone will call the build plugin to build the image, tag with the combination of the number of the build, the branch and 6 digits from the hash of the commit. This enables traceability for the image in production. This tagged image is then pushed to Harbor to become available for the next steps.
- Security Test: While in Harbor, Clair (our choice for security scan) will inspect the image for vulnerabilities. We've set an auto-scan on push and set a rule to prevent vulnerable images with low severity and above to be pulled. This constraint will force the pipeline to fail in further steps in case of vulnerabilities.
- Helm: The next step is to build end publish the helm chart that will be used in deploy. We used the helm package and helm push commands in a custom docker image to execute this step. As one of our principles is to reduce complexity and redundancies in configuration, we maintain one repo for templating all helm charts. The developer must only describe the specific configs for the service in a values.yaml file inside the project and the script will blend it with the unified template to generate the chart. Our chart versioning policy is to match the minor version of a chart (ex: 0.1.4) for each image build (ex: 4-master-abcdef).
- Deploy Test Environments: In this step, drone invokes the drone-helm3 plugin to deploy the helm chart on each environment. This plugin can be configured to set values during the deploy. This is quite useful for setting specific URLs for ingress according to the environment.
- Deploy Staging and Production: Despite the name, drone will only configure the Argo application to sync the new helm chart. Argo will be the one to actually deploy and enforce configuration on staging and production kubernetes environments. What drone does is check if the Argo application exists (in case of a new repo) and create it otherwise. It sets some helm values just like the last step and sets staging deployment as auto-sync and production as manual sync.
- Authorization: This is not a drone step. The authorization occurs when the Release Manager approve the deployment in production and manually sync all "out-of-syn" repos in Argocd.
![Alt text](https://github.com/alexismaior/cicd/blob/master/argocd.png?raw=true "ArgoCD")
  In Argo, it is possible not only to sync specific kubernetes objects but also check the history of all deployments and execute rollbacks for previously working releases.
