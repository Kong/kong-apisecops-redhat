# kong-apisecops-redhat

## Infrastructure

Deploy the Openshift Infrastructure

```console
ansible-playbook ansible/playbook.yaml --extra-vars "konnect_email=<yourEmail> konnect_pass=<yourPassword> konnect_controlPlane=<your-runtime-group-id>"
```

## Tutorial

### Step 1

Dev Team Persona - APISpec Design is complete, pr to security team for review and deploy to sandbox environment.

Execute Pipeline 1:

```console
oc create -f run/1-disputes-pipeline-run.yaml
```

### Step 2

APIOps Persona - automatically validate the new apispec, and deploy to sandbox for teams to begin discovering and developing aginst

Exuecte Pipeline 2:

```consoles
oc create -f run/2-apiops-sandbox-pipeline-run.yaml
```

### Step 3

Dev Persona - I am ready to let teams test the working API, I will deploy the dev, and request security to deploy the api to dev.
