# CI-CD Overview

Two Openshift projects are required to deploy our applications.
The `app-dev-ci-cd` project has the Jenkins server and manages the build process for our images. Built images are pushed into the `consultant-360` project for deployment.

![CI_CD_for_Feedback_360](uploads/c17ec19e8ab44c271385e07f3199b1cb/CI_CD_for_Feedback_360.png)

# Steps to deploy a single app to the cluster

Before we start, log in to the target cluster with the `oc` CLI tool. 

The cluster for the Consulting360 project is:
https://console-openshift-console.apps.shared-dev.dev.openshift.opentlc.com

## Clone required repos

In a fresh directory, clone the `infrastructure` and `app-dev-openshift-ci-cd` repos.

`app-dev-openshift-ci-cd` has ansible playbooks for creating the jenkins pipelines.

```
git clone ssh://git@gitlab.consulting.redhat.com:2222/appdev-coe/app-dev-openshift-ci-cd.git
```

`infrastructure` has ansible playbooks for creating the `consultant-360` projects and the application templates.  

```
git clone ssh://git@gitlab.consulting.redhat.com:2222/appdev-coe/consultant-360-feedback/infrastructure.git
```

## Install the Ansible openshift-applier

```
cd app-dev-openshift-ci-cd
ansible-galaxy install -r requirements.yml --roles-path=roles
```

## Run ansible playbook for app-dev-openshift-ci-cd

**Skip this section if you are deploying to the existing projects in our cluster -- you will not have sufficient permissions to execute some of the steps in the playbooks.** 

Run `bootstap` to create the app-dev-ci-cd project and set policies
```
ansible-playbook site.yml -i inventory/hosts -l bootstrap
```

Run `ci-cd-tooling` to install jenkins, sonarqube, and other supporting services.
The `-e "nexus_validate_certs=false"` is required due to a kubernetes bug
```
ansible-playbook site.yml -i inventory/hosts -l ci-cd-tooling -e "nexus_validate_certs=false"
```
## Mac Users!

> **WARNING:\_** A recent update has placed a limit of the number of forks a process can create. To disable this, set **OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES**

##### Example

```
OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES ansible-playbook site.yml -i inventory/hosts -l consultant360 -e "include_tags=survey-openapi"
```
## Run ansible playbook for infrastructure

```
cd -
cd infrastructure
```
**Skip this command if you are deploying to our existing projects in the cluster**
```
# run bootstap to create the project and set policies
ansible-playbook site.yml -i inventory/hosts -l bootstrap
```

If you are deploying ALL the services for consulting-360, run:
```
ansible-playbook site.yml -i inventory/hosts -l consultant360 
```

To deploy a specific service, run:
```
ansible-playbook site.yml -i inventory/hosts -l consultant360 -e "include_tags=SERVICE_NAME"
``` 
where `SERVICE_NAME` is the tag name for that service:

```
health-db-deploy
survey-spring-service
survey-node-service
survey-openapi
manager-front

```

**IMPORTANT** If your service requires database access, you MUST deploy the database service first. 

```
ansible-playbook site.yml -i inventory/hosts -l consultant360 -e "include_tags=health-db-deploy"
```

# Manually building and deploying

## start the build

```
# switch oc client to the app-dev-ci-cd project
oc project app-dev-ci-cd

# view available builds. After running the infrastructure ansible playbooks, you should see the build config for your service(s) here
oc get build

# build it!
oc start-build bc/<build-config-for-app>
```

To view Jenkins progress of the build, get the logs and follow the Jenkins url:
```
$ oc logs -f bc/survey-service-pipeline
info: logs available at /https://jenkins-app-dev-ci-cd-keith.apps.shared-dev.dev.openshift.opentlc.com/blue/organizations/jenkins/app-dev-ci-cd-keith%2Fapp-dev-ci-cd-keith-survey-service-pipeline/detail/app-dev-ci-cd-keith-survey-service-pipeline/9/
```

## deployment

```
oc project consultant-360-dev
oc rollout latest dc/<name-of-deployment-config-for-app>
```