# Blockchain CI/CD pipelines & tasks

This repo contains Tekton pipelines and tasks for implementing CI/CD for Hyperledger Fabric on IBM Blockchain Platform chaincode using the install & instantiate chaincode lifecycle from prior to v2.0.

## Overview

Hyperledger Fabric chaincode does not have inherent support for CI/CD and similarly is not supported by ArgoCD. The pipelines, tasks and repos here are designed to replicate the Gitops approach and apply it to chaincode. The high level steps are as follows:

1. Developer makes changes to chaincode and pushes the changes to the Git repo.
1. `chaincode-ci-pipeline` is triggered which tests & packages the chaincode before updating the version for deployment in the `gitops` repo.
   1. Pushes packaged chaincode to `packages` repo.
   1. Pulls `gitops` repo and updates version to deploy to be the one just packages.
1. `chaincode-cd-pipeline` is triggered which installs the chaincode on the given peers and instantiates it on the given channel.
   1. Pulls `gitops` and `packages` repos.
   1. Uses `install.yaml` playbook from `packages` repo to install version desired in `gitops` repo.
   1. Uses `instantiate.yaml` playbook from `packages` repo to instantite chaincode.

## How to use

1. Update the values in `./config/peers-config` for the IBM Blockchain Platform API endpoint, key and secret, as well as the channel name and peer details.
1. Create a config map for the configuration: `oc create configmap <configmap_name> --from-file="./config/peers-config.yaml"`
1. Create secrets in the cluster for the admin identities of organisations in the network: `oc create secret <secret_name> --from-file="./config/<org_admin_identity_file>"`
1. Update/add the configmap and secret names to the `volumes` and `volumeMounts` in both `install-chaincode.yaml` and `instantiate-chaincode.yaml` Tekton tasks.
1. Push the `gitops` and `packages` folders to separate Git repos.
1. Update the params values in `./tekton/TriggerTemplates/chaincode-ci-trigger-template.yaml` and `./tekton/TriggerTemplates/chaincode-cd-trigger-template.yaml`

## Limitations

- There is no way to query a Fabric channel for currently installed/instantiated version. This means that if the chaincode install succeeds but the instantiate fails for some reason, then re-running the pipeline may fail because the version to install is already installed. Sometimes manual intervention is required to resolve.
