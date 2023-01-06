# APISecOps Demo on RedHat OpenShift - with Insomnia, Kong Konnect and Tekton on ROSA

Here at Kong APISecOps centers around four core fundmentals:

* **Centralization** - centralize API Management to a single control plane. Irrespective of cloud provider, or platform, all can be managed from the same control plane.

* **Governance** - a governance team should be able easily customize API linting for security concerns and quickly validate.

* **API Design First** - Development Team should design and document the API upfront to validate they are update to date with current governance requirements, and accurate documentation.

* **GitOps** - The API Spec, supporting documentation, governance, and API administrative should all be handled via gitops best practices for speed, reslience, and reliablity in the process.

The objective of this demo is to showcase how to streamline API management with the above APISecOps best practices in mind on Red Hat OpenShift.

<img src="img/arch.png" alt="kong apisecops apiops rosa"/>

## Prequisites

1. **Openshift Cluster** - This demo will step through the rosa cli command to create a ROSA cluster but any OpenShift cluster will suffice. This demo has been tested on 4.11.

2. **Ansible Core >= 2.13** -  The playbooks have been tested on 2.13.5 and python version 3.10.8. More information can be found at [Installing Ansible][Ansible_Install_Distros].

3. **Kong Konnect Plus Account** - The demo requires and *Plus* grade account because 2 runtime groups and several enterprise grade plugins. For more information please review the [Kong Konnect Pricing Plan][Konnect_Pricing].

4. **Insomnia** - To download check out [Insomnia Download][Insomnia_Install].

## Project Directory Overview

```console
├── README.md 
├── ansible                           <-- ansible scripts
│   ├── disputes
│   ├── playbook-uninstall.yaml
│   ├── playbook.yaml
│   ├── tasks
│   └── vars
├── konnect                           <-- konnect deck configuration, backup just in case need to re-align konnect runtime group configuration
│   ├── deck-default-rg.yaml
│   ├── ...
│   └── deck-disputes-dev-rg.yaml
└── run                               <-- tekton pipeline runs
    ├── apiops-dev-pipeline-run.yaml
    ├── apiops-sandbox-pipeline-run.yaml
    └── disputes-pipeline-run.yaml
```

## Deploy Infrastructure

### ROSA

Create System Variables:

```console
CLUSTER_NAME=<my-cluster-name>
REGION=<my-aws-region>
```

Create a small ROSA cluster:

```console
rosa create cluster --cluster-name=$CLUSTER_NAME --region=$REGION --multi-az=false --compute-machine-type=t3.xlarge
```

When the Cluster install is complete, create a cluster-admin user:

```console
rosa create admin --cluster $CLUSTER_NAME
```

Validate you can login to the cluster via the credentials provided by the rosa cli stdout. Once login is successful you can proceed to the next step.

### ROSA and Konnect Configuration

Execute the install ansible playbook. The play will do the following:

* Cert Manager Operator - install and create Konnect DP self-signed certs
* Openshift Pipelines Operator - install
* Gitea - install and configure
* Konnect
  * create and/or configure runtime groups (Default and Dev)
  * create konnect dataplanes (runtime instances)
* Create APIOps Namespaces, install Tekton Pipelines and create Tekton Pipeline Runs
* Deploy Disputes Sample Application

```console
ansible-playbook ansible/playbook.yaml --extra-vars "konnect_email=<yourEmail> konnect_pass=<yourPassword>"
```

Any required information, urls, dummy passwords, load balancers, are spit out as the last task in the ansible playbook, and saved in the `ansible/demo_facts.json` file for safe keeping.

## Devops Tutorial


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

## References

* [Kong Konnect Pricing Plan][Konnect_Pricing]
* [Installing Ansible][Ansible_Install_Distros]
* [Rosa Quickstart Guide][Rosa_Docs]
* [Install Insomnia][Insomnia_Install]

    [list of links]: #

[Konnect_Pricing]: https://konghq.com/pricing
[Ansible_Install_Distros]: https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html
[Rosa_Docs]: https://docs.openshift.com/rosa/rosa_getting_started/rosa-quickstart-guide-ui.html
[Insomnia_Install]: https://insomnia.rest/download