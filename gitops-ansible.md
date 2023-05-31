## Ansible Automation Platform

The Ansible Automation Platform (AAP) allows you to create automation content in a more consistent way, manage automation processes more effectively, and scale automation capacity easily and on-demand. AAP 2 decouples the control plane and execution plane.

## AWX Resource Operator

The AWX Resource Operator provides the ability to launch jobs and create Ansible job templates by creating resources directly in a Kubernetes/OpenShift environment. There are currently two custom resources provided by the AWX Resource Operator: ansible job, which launches a job in the AWX/AAP instances, and jobtemplate, which creates a template in AAP.

## GitOps with ArgoCD

GitOps with ArgoCD ensures consistency in applications when deploying to different cluster environments, such as development, staging, and production. RH GitOps uses ArgoCD as a controller to maintain cluster resources.

Workflow

The workflow looks like this :

Pull request —> Git repository —> Openshift GitOps —> AWS resource Operator —> Ansible automation platform. 

### How this could be useful to us :

We could use the workflow to automate running playbooks based on events or before upgrading clusters. (Automatically launching the health check playbook before an upgrade or maintenance)

Usefulness mitigated by TeamCity but still cool to know this exists.