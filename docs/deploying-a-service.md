# CI-CD Overview

Two Openshift projects are required to deploy our applications.
The `appdev-example-ci-cd` project has the Jenkins server and manages the build process for our images. Built images are pushed into the `dev`, `test` and `demo` projects for deployment.


# Steps to deploy a single app to the cluster

Before we start, log in to the target cluster with the `oc` CLI tool. 

If you need access to a cluster, create a temporary 4.x cluster at [https://labs.opentlc.com](OpenTLC).

## Clone this repository

In a fresh directory, clone the `infrastructure` repository.

`infrastructure` has ansible playbooks for creating the example projects and the application templates.  

```
git clone https://github.com:2222/redhat-appdev-practice/infrastructure.git
```

## Install the Ansible openshift-applier

```
cd infrastructure
ansible-galaxy install -r requirements.yml -p roles
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
ansible-playbook site.yml -i inventory/ -l bootstrap
```

If you are deploying ALL the services for consulting-360, run:
```
ansible-playbook site.yml -i inventory/ 
```

To deploy a specific service, run:
```
ansible-playbook site.yml -i inventory/ -l consultant360 -e "include_tags=SERVICE_NAME"
``` 
where `SERVICE_NAME` is the tag name for that service:

```
survey-spring-service
survey-node-service

```

To deploy to a different namespace, run:
```
ansible-playbook -i inventory/ site.yml -e "namespace_prefix=mynamespace"
```

# Manually building and deploying

## start the build

```
# switch oc client to the app-dev-ci-cd project
oc project appdev-example-ci-cd

# view available builds. After running the infrastructure ansible playbooks, you should see the build config for your service(s) here
oc get build

# build it!
oc start-build bc/<build-config-for-app>
```

To view Jenkins progress of the build, get the logs and follow the Jenkins url:
```
$ oc logs -f bc/survey-service-pipeline
```

## deployment

```
oc project appdev-example-dev
oc rollout latest dc/<name-of-deployment-config-for-app>
```