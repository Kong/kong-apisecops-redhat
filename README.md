# kong-apisecops-redhat

## Infrastructure

Deploy the Openshift Infrastructure

```console
ansible-playbook ansible/playbook.yaml --extra-vars "konnect_email=<yourEmail> konnect_pass=<yourPassword>"
```

## Tutorial

### Step 1

Dev Team Persona - APISpec Design is complete, pr to security team for review and deploy to sandbox environment.

Execute Pipeline 1:

```console
oc create -f run/disputes-pipeline-run.yaml
```

### Step 2

APIOps Persona - automatically validate the new apispec, and deploy to sandbox for teams to begin discovering and developing aginst

Execute Pipeline 2:

```console
oc create -f run/apiops-sandbox-pipeline-run.yaml
```

As an APIOps Person - I will have a PR approval process protecting my branches. In the case of the demo an APIOperator will manually merge in the pr.

### Step 3

Dev Persona - I am ready to let teams test the working API, I will deploy the dev, and request security to deploy the api to dev.

Execute Pipeline 3:

```console
oc create -f run/apiops-dev-pipeline-run.yaml
```
