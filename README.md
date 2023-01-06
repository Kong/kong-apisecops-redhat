# APISecOps Demo on RedHat OpenShift - Insomnia, Kong Konnect, Tekton on ROSA

Here at Kong APISecOps centers around four core fundmentals:

* **Centralization** - entralize API Management to a single control plane. Irrespective of cloud provider, or platform, all can be managed from the same control plane.

* **Governance** - a governance team should be able easily customize API linting for security concerns and quickly validate.

* **API Design First** - Development Teams should design and document the API upfront to validate they are update to date with current governance requirements, and accurate documentation.

* **GitOps** - The API Spec, supporting documentation, governance, and API administrative should all be handled via gitops best practices for speed, reslience, and reliablity in the process.

The objective of this demo is to showcase how to streamline Kong API management with the above APISecOps best practices in mind with Kong in the Red Hat Openshift Ecosystem.

## Tutorial Overview

First, a brief overview of the core infrastructure laid down.

<img src="img/arch.png" alt="kong apisecops apiops rosa"/>

*Note - just to clarify this demo is run within 1 cluster, but in order to clearly depict the tekton pipeline ci/cd and the infra it is depicted as two. It's really just all in the same cluster.*

**Konnect**

Two Runtime Groups will be either created or at least checked that it exists - Default, and Dev. Each runtime group will be provisioned 1 runtime instance (also referred to as a Gateway, Dataplane, or Proxy), each one will be in their own namespace, kong-sandbox, and kong-dev. These Gateways are exposed via loadbalancers, and are where API Consumers can call the protected backend services.

**Openshift Pipelines/Tekton**

There are three tekton pipelines:

1. **disputes-apispec-review pipeline** - will open a pr to push apispec to konnect-sandbox runtime group.
2. **api-gateway-sandbox-pipeline** - review open prs on sandbox, execute governance tests, and api administration to the konnect sandbox runtime group.
3. **api-gatway-dev-pipeline** - will open pr to publish apispec to dev runtime group and Konnect Service Hub for Dev Portal Integration.

**Gitea (Self-hosted Git service)**

Gitea is a self-hosted Git service. It is stood up in the cluster in the `gitea` namespace. Two the git repos required to run the demo are imported, and any dummy passwords needed for the demo are seeded in the projects and provided to the user. This is just to minimize external dependencies.

**Disputes Sample Application**

The sample application is deployed in `disputes-dev` namespace. It is a very small JBoss EAP application server.

**Diagram**

The diagram below a high level architecture overview of the pipeline-to-infrastucture setup summarizing what was just discussed. 

## Prequisites

1. **Openshift Cluster** - This demo will step through the rosa cli command to create a ROSA cluster but any OpenShift cluster will suffice. This demo has been tested on OCP 4.11.

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

* **Cert Manager Operator** - install and create Konnect DP self-signed certs
* **Openshift Pipelines Operator** - install
* **Gitea** - install and configure
* **Konnect**
  * create and/or configure runtime groups (Default and Dev)
  * create konnect gateways (runtime instances)
* **APIOps** - create namespaces, install tekton pipelines and create tekton pipelineruns
* **Disputes Sample App** - create namespace and deploy

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