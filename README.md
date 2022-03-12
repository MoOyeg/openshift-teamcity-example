# openshift-teamcity-example

## Status: Not Ready

This repo provides an example of how to run the [Teamcity CI Server](https://www.jetbrains.com/help/teamcity/teamcity-documentation.html) on Openshift.
Repo will attempt to spin up the TeamCity CI Server and show an example to build an image and push it into the internal Openshift Image Registry.Repo also installs the TiDB Operator to serve as the database for the TeamCity Server.This is a general example and if being deployed in production hardening would be required for security.

- See Known Issues Below for Known Issues

## Pre-Requisites

- Repo leverages Openshift GitOps to perform Application Deployment.[Install Openshift Gitops](https://docs.openshift.com/container-platform/4.9/cicd/gitops/installing-openshift-gitops.html).

- Repo requires a [dynamic default storage class](https://docs.openshift.com/container-platform/4.9/storage/dynamic-provisioning.html#basic-storage-class-definition_dynamic-provisioning)

- Repo requires elevated privileges to run.

### Steps

- Git Clone this repo

- Change directory into this repo

- Create infra application via ArgoCD  
  `oc create -f ./deploy-tooling/argocd-deploy/application-infra.yaml`  
  This should create the necessary namespaces, serviceaccount and permission to run example.

- The ServiceAccount being used will require enhanced privileges and the gitops tool might not have those permissions. Create the scc for the teamcity-sa service account.  
  `oc create -f ./deployables/Infra/base/teamcity-scc-anyuid.yaml`

- Create TiDb Application via ArgoCD  
  `oc create -f ./deploy-tooling/argocd-deploy/application-TiDB.yaml`  
  This should create the the TiDB Operator,TiDB MySQl Cluster, create a user called teamcity a database called teamcity and set the root and teamcity user passwords to teamcity.

- Create the TeamCity Server via ArgoCD
  `oc create -f ./deploy-tooling/argocd-deploy/application-teamcity.yaml`

- Wait for teamcity deployment to be ready
  `oc rollout status deployment/teamcity-server-instance -n teamcity-server -w`

- Get Teamcity Server URL  
  `oc get route/teamcity-server-instance -n teamcity-server -o jsonpath='{.spec.host}'`

- Open TeamCity URL in Browser, this should show you an empty database page like below,like below.Look through known issues if different.  
  ![TeamCity Empty DB Start Page](./assets/teamcity_empty_start.png?raw=true "TeamCity Empty DB Start Page") Select "i'm a server administrator...link"

- Link will bring up a prompt for a Token.Token can be found in the pod deployment log. Look for the deployment instance , the running pod and then open the logs for the pod. The token should be visible in lower end of logs.See Sample belows.  
  ![TeamCity Enter Token](./assets/teamcity_enter_super_user_token.png?raw=true "TeamCity Enter Token")

  ![TeamCity Deployment Pod](./assets/teamcity_deployment_pod.png?raw=true "TeamCity Deployment Pod")

  ![TeamCity Super User Token](./assets/teamcity_super_user_token.png?raw=true "TeamCity Super User Token") Enter Token Value from logs into the entry box.

- Teamcity should start initilizing.

- Teamcity should show a License Agreement Box. If license is ok accept and continue.

- Teamcity should display a login page. Select the option to Login as Super User and provide the Token copied from logs above.
  ![TeamCity Login Token](./assets/teamcity_login_empty.png?raw=true "TeamCity Login Token")

  ![TeamCity Token](./assets/super_user_token.png?raw=true "TeamCity Token")Enter Token Value from logs into the entry box.

- You should be in the Teamcity HomePage after Login.TeamCity Server should have pre-configured Project, Build and Profiles configured via the Restore of backup Files. Unfortunately credentials are not automatically updated yet, so we need to update them.

  Homepage
  ![TeamCity Home Page](./assets/teamcity_homescreen.png?raw=true "TeamCity Home page")

  Select the Testflask project Page and then Click Edit Settings on Top Right of Page.
  ![TestFlaskProject](./assets/testflask_project.png?raw=true "TestFlask project")

  You should arrive on the Settings Page.Select the Cloud Profiles option.
  ![TestFlaskProject Settings](./assets/testflask_settings.png?raw=true "TestFlask project Settings")

  In the Cloud Profiles list you should find the pre-configured cloud profile for "Openshift".Select the edit button for that cloud profile.
  ![TestFlask Cloud Profile](./assets/testflask_cloud_profile.png?raw=true "TestFlask Cloud Profile")

  The edit cloud profile should look like below
  ![Openshift Cloud Profile](./assets/testflask_edit_cloud_profile.png?raw=true "Openshift Cloud Profile")

  We need to replace our token, get the teamcity-sa token  
  TOKEN=`oc exec $(oc get pods -n teamcity-server -l app=teamcity-server-instance -o name) -n teamcity-server -- oc whoami -t`

  Copy value from above token and replace in token field. Test Connection to make sure it works and choose the option to save.

  Scroll down to the Agent Images on the same page and select the edit option.Make sure the Deployment Option is set to teamcity-agent-instance which we created earlier and click save this should spin up an agent for us.If it does not go to the agents link(top page) and spin one up.
  ![Teamcity Agent Profile](./assets/teamcity_agent_image.png?raw=true "Teamcity Agent Profile")

  With the agent running we can go back to our testFlask project page and select run on the build.
  ![TestFlaskProject](./assets/testflask_project.png?raw=true "TestFlask project")

## Known Issues

- ArgoCD Status for TiDB Operator is wrong so TiDB Application shows up as degraded. Working on a Fix.

- If the Docker agent shows a "docker.server.version exists" it means the teamcity-sa does not have the necessary permissions.
