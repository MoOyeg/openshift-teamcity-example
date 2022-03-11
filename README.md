# openshift-teamcity-example

## Status: Not Ready

This repo provides an example of how to run the [Teamcity CI Server](https://www.jetbrains.com/help/teamcity/teamcity-documentation.html) on Openshift.
Repo will attempt to spin up the TeamCity CI Server and show an example to build an image and push it into the internal Openshift Image Registry.Repo also installs the TiDB Operator to serve as the database for the TeamCity Server.This is a general example and if being deployed in production hardening would be required for security.

## Pre-Requisites

- Repo leverages Openshift GitOps to perform Application Deployment.[Install Openshift Gitops](https://docs.openshift.com/container-platform/4.9/cicd/gitops/installing-openshift-gitops.html).

- Repo requires a [dynamic default storage class](https://docs.openshift.com/container-platform/4.9/storage/dynamic-provisioning.html#basic-storage-class-definition_dynamic-provisioning)

- Repo requires elevated privileges to run.

### Steps

- Git Clone this repo

- Change directory into this repo

- Create infra application via ArgoCD  
  `oc apply -f ./deploy-tooling/argocd-deploy/application-infra.yaml`  
  This should create the necessary namespaces, serviceaccount and permission to run example.

- The ServiceAccount being used will require enhanced privileges and the gitops tool might not have those permissions. Create the scc for the teamcity-sa service account.  
  `oc apply -f ./deployables/Infra/base/teamcity-scc-anyuid.yaml`

- Create TiDb Application via ArgoCD  
  `oc apply -f ./deploy-tooling/argocd-deploy/application-TiDB.yaml`  
  This should create the the TiDB Operator,TiDB MySQl Cluster, create a user called teamcity a database called teamcity and set the root and teamcity user passwords to teamcity.

- Create the TeamCity Server via ArgoCD
