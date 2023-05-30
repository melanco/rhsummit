# Heading to production with RHOCP Service Mesh

Heading to production with RHOCP Service Mesh

Speakers :

Ortwin Schneider

Jamie Longmuir

6 modules :

Module 1 : Designing service mesh premieter and operating model

module2 : Service mesh deployment en observilability stack

module 3 : set up travel demo servicemeshCOntrolPlane for production

Module 4 : securting gateways

Module 5 : Business authZ restrctions and corporate CA setup

Module 6 : Sevice Mesh day 2 operations

[https://github.com/skoussou/summit-2023-ossm-labs](https://github.com/skoussou/summit-2023-ossm-labs)

---

# Heading to Production Lab Scenario

## ABOUT THIS LAB

This lab will take you through the use case of the *Travel Agency* organization which intends to utilize Service Mesh for its *Travel Demo* application. You will be introduced to techniques of designing a mesh based on real stakeholder requirements and delivering against those from development all the way to production. In addition, it introduces a set of corporate personas with responsibilities and permissions to create and operate the mesh.

## LAB ASSETS

Before you begin, access your [code-server environment](https://codeserver-codeserver-user25.apps.cluster-qnkf6.sandbox671.opentlc.com/) and ensure you have access to the lab assets `summit-2023-ossm-labs`.

Open a terminal (**`Terminal`** → **`New Terminal`** from the menu) in the *code-server* so that you can execute the commands during the labs.

Although for the labs the assets are made available you can additionally get and inspect these resources at a later stage from the [summit-2023-ossm-labs](https://github.com/skoussou/summit-2023-ossm-labs) github repository.

## Use of correct Openshift Namespaces

|  | You will be using a shared cluster and the Service Mesh users are also common with access to multiple namespaces. Therefore, it is very important each participant uses their allocated namespaces. In order to make use of the correct namespaces make sure you are accessing those with the allocate user id user25 |  |
| --- | --- | --- |

As part of a Kick-Off meeting the Travel Agency stakeholders have stated the capabilities they would like to benefit from with the introduction of Service Mesh:

1. The *Development Team* wants to be able to trace (every) request during development, and a sample of 20% of traffic in Production
2. The *Product Team* wants to see metrics around performance/usage and storing them for up to 1 week
3. The *Security* team wishes to enable mTLS in all service-to-service communications
4. The *Platform Team* wants to centrally manage all security aspects
5. The *Development Teams* wants to implement some resiliency to make the overall application more stable and reliable.

## User Governance Model

**The personas, roles and responsibilities are greatly affected by certain organizational, operational and governance choices around the cloud application platforms.** The Travel Agency company considered in the KickOff meeting the following aspects:

***Type of cluster***Is it multi tenant cluster running/operating several application domains or is it a dedicated cluster for one app domain? And the topology of the Service Mesh (multi-tenant mesh vs. a single cluster mesh).***Choices of automation***What kind of automation to use to configure apps, the mesh configuration and the cloud platform? (Using i.e. CI/CD Pipelines, GitOps, Ansible, Scripting, ACM or none).***Platform (Service Mesh) Operating Model***Is it a producer-consumer platform where admins/ops deploy all configurations and developers consume or is it a self-service platform?***Dev(Sec)Ops Methodology***Whether the team have adopted [DevSecOps approaches](https://www.redhat.com/en/topics/security/devsecops/approach) for application and cloud configuration delivery. DevSecOps approaches bring together development, security, and operations into a collaborative shared-responsibility paradigm. The goal is to break down barriers between roles, disciplines, and teams across an organization to encourage collaboration and work toward common goals. A DevSecOps approach encompasses people, processes, technology, and governance.

For the purpose of the provided scenarios the Travel Agency has selected the following options determining the *Model of Operation*.

| # | Strategy | Option |
| --- | --- | --- |
| 1 | Cluster Type | Dedicated Cluster |
| 2 | Automation | Scripting (Step2 is GitOps) |
| 3 | Platform Operating Model | https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m1/walkthrough.html?LAB_PARTICIPANT_ID=user25&OCP_DOMAIN=apps.cluster-qnkf6.sandbox671.opentlc.com&API_URL=https%3A%2F%2Fapi.cluster-qnkf6.sandbox671.opentlc.com%3A6443#sidenote2 |
| 4 | Dev(Sec)Ops Methodology | not covered by the Labs |

Upstream Istio, and so OpenShift Service Mesh, do not define standard or default user roles it is up to each project/implementation to define the appropriate permissions and roles for the required personas in order to access the necessary Service Mesh resources. We have defined those based on the *Model of Operation* and expected User Governance.

In the following table you’ll see the Persona / Role mappings for the Travel Agency personas:

| Persona | Role | Responsibilities |
| --- | --- | --- |
| Platform Admin | Cluster Admin (Default OpenShift cluster-admin role) | Owner of multiple OpenShift clusters, deploys operators and sets organizational policies. |
| Mesh Operator | https://github.com/skoussou/summit-2023-ossm-labs/blob/helm/helm/bootstrap/templates/clusterroles.yaml#L5 | Operates parts of the cluster and the domain based service mesh for the hosted services. Creates and operates Service Mesh tenants. |
| Domain Owner (Tech Lead) | https://github.com/skoussou/summit-2023-ossm-labs/blob/helm/helm/bootstrap/templates/clusterroles.yaml#L142 | Responsible for an Application Domain. Onboards developers in the team and understands inter/intra service dependencies. Creates and configures Service Mesh resources (VirtualService, DestinationRule etc.) in their domain. |
| Developer | https://github.com/skoussou/summit-2023-ossm-labs/blob/helm/helm/bootstrap/templates/clusterroles.yaml#L275 | Develops services in his Application Domain. Needs to be kept aware of the health, performance and functional correctness of the solution. |
| Application Ops Team | https://github.com/skoussou/summit-2023-ossm-labs/blob/helm/helm/bootstrap/templates/clusterroles.yaml#L142 | The Application Ops team monitors and maintains the running applications in the deployed cluster and within the domain hosted mesh (OSSM tenant), including extracting logs, executing commands to verify state, and troubleshooting in higher (non-development) environments |
| Product Owner | https://github.com/skoussou/summit-2023-ossm-labs/blob/helm/helm/bootstrap/templates/clusterroles.yaml#L275 | The Product Owner needs to be aware of the health, usage, cost as well as other metrics around the business domain of the solution. |

The `Mesh Operator`, `Mesh Application Viewer` and `Mesh Developer` Roles have been pre created for this Lab and `Rolebinding` has been added for each user (see links on the table below).****

For this Lab all the required OpenShift users for the identified personas have been pre-created and mapped to the corresponding roles.<

| emma | Mesh Operator | https://github.com/skoussou/summit-2023-ossm-labs/blob/helm/helm/ossm/templates/dev/rolebindings-emma.yaml | dev-istio-system |
| --- | --- | --- | --- |
| cristina | Travel Portal Domain Owner (Tech Lead) | https://github.com/skoussou/summit-2023-ossm-labs/blob/helm/helm/ossm/templates/dev/rolebindings-cristina.yaml | dev-travel-portal, dev-travel-control |
| farid | Travel Services Domain Owner (Tech Lead) | https://github.com/skoussou/summit-2023-ossm-labs/blob/helm/helm/ossm/templates/dev/rolebindings-farid.yaml | dev-travel-agency |
| john | Developer (TP) | https://github.com/skoussou/summit-2023-ossm-labs/blob/helm/helm/ossm/templates/dev/rolebindings-john.yaml | dev-travel-portal, dev-travel-control |
| mia | Developer (TS) | https://github.com/skoussou/summit-2023-ossm-labs/blob/helm/helm/ossm/templates/dev/rolebindings-mia.yaml | dev-travel-agency |
| mus | Product Owner | https://github.com/skoussou/summit-2023-ossm-labs/blob/helm/helm/ossm/templates/dev/rolebindings-mus.yaml | dev-travel-portal, dev-travel-control, dev-travel-agency |

## Development User credentials

| username | password | Enterprise Persona |
| --- | --- | --- |
| emma | emma | Mesh Operator |
| cristina | cristina | Travel Portal Domain Owner (Tech Lead) |
| farid | farid | Travel Services Domain Owner (Tech Lead) |
| john | john | Developer (TP) |
| mia | mia | Developer (TS) |
| mus | mus | Product Owner |

**You do not have to execute anything in this part of the lab** as lab instructors have **pre-setup all the required assets for the Travel Agency development environment**, including users, roles, namespaces, apps, service mesh etc. You will create the PROD environment step-by-step in the next scenario. You can inspect at the end of the lab the installation of the development environment by exploring the `ArgoCD` resources used to [Install & Setup the Development Environment](https://github.com/skoussou/summit-2023-ossm-labs/tree/helm/helm/ossm/templates/dev)

Reading the following is totally optional (you are urged to move to *Part II* as soon as possible) as we only briefly describe the steps the [Travel Agency Personas](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m1/walkthrough.html?LAB_PARTICIPANT_ID=user25&OCP_DOMAIN=apps.cluster-qnkf6.sandbox671.opentlc.com&API_URL=https%3A%2F%2Fapi.cluster-qnkf6.sandbox671.opentlc.com%3A6443#_travel_agency_personas_roles) would have had to follow in order to deliver such a development environment:

### **Actions with role Cluster Admin**

1. As openshift administrator (role: `Cluster Admin`) add the `Service Mesh` operators in the OCP Cluster
2. As openshift administrator (role: `Cluster Admin`) once the operators have been successfully installed, create the necessary *Travel Agency* Namespaces
3. As openshift administrator (role: `Cluster Admin`) create the `Service Mesh Roles`
4. As openshift administrator (role: `Cluster Admin`) create the Service Mesh Users and assign Roles

### **Actions with role Mesh Operator**

1. As **emma** (role: `Mesh Operator`) create the `Service Mesh` controlplane namespaces and the `ServiceMeshControlPlane (SMCP)` resource.

### **Actions with role Mesh Developer**

1. As **farid** (role: `Mesh Developer`) *Travel Services Domain Owner (Tech Lead)*
    ◦ Onboard namespace `dev-travel-agency` to Service Mesh `dev-basic` by adding a `ServiceMeshMember` (`SMM`) resource in `dev-travel-agency`.
    ◦ Deploy the Applications in `dev-travel-agency` namespaces
2. As **cristina** (role: `Mesh Developer`) *Travel Portal Domain Owner (Tech Lead)*
    ◦ Onboard namespaces `dev-travel-control` and `dev-travel-portal` to Service Mesh `dev-basic` by adding a `ServiceMeshMember` (`SMM`) resource in each namespace for `dev-basic` membership.
    ◦ Deploy the Applications in `dev-travel-control`, `dev-travel-portal` namespaces and `Istio` configs to expose Service Mesh services.

### **Final Actions with role Mesh Operator**

1. As **emma** (role: `Mesh Operator`) create the Istio `Gateway` resource

`OSSM` -like Istio- now offers the ability for the injection of an ***ingress Gateway*** in the dataplane however for the *Travel Agency* use case the Architects have selected a ***Self-Service(Restricted)*** [User Governance Model](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m1/walkthrough.html?LAB_PARTICIPANT_ID=user25&OCP_DOMAIN=apps.cluster-qnkf6.sandbox671.opentlc.com&API_URL=https%3A%2F%2Fapi.cluster-qnkf6.sandbox671.opentlc.com%3A6443#_user_governance_model) where the `Mesh Operator` will be responsible for ingress/egress resource configurations.

Export the following in a terminal in your [code-server environment](https://codeserver-codeserver-user25.apps.cluster-qnkf6.sandbox671.opentlc.com/).

- export CLUSTER_API=https://api.cluster-qnkf6.sandbox671.opentlc.com:6443
- export LAB_PARTICIPANT_ID=user25
- export OCP_DOMAIN=apps.cluster-qnkf6.sandbox671.opentlc.com
- export SSO_CLIENT_SECRET=bcd06d5bdd1dbaaf81853d10a66aeb989a38dd51

### **Task 1: Access the Travel Control Dashboard**

- Log in via CLI with user `emma` who has access to `user25-dev-istio-system` and get the URL to the application business dashboard. Paste the URL in a Browser to verify that you can reach the **Travel Control Dashboard** .

`./lab-3/login-as.sh emma
echo "http://$(oc get route istio-ingressgateway -o jsonpath='{.spec.host}' -n user25-dev-istio-system)"`

~/summit-2023-ossm-labs $ ./lab-3/login-as.sh emma

Logging in with
oc login -u emma -p emma [https://api.cluster-qnkf6.sandbox671.opentlc.com:6443](https://api.cluster-qnkf6.sandbox671.opentlc.com:6443/)
The server uses a certificate signed by an unknown authority.
You can bypass the certificate check, but any data you send to the server could be intercepted by others.
Use insecure connections? (y/n): y

WARNING: Using insecure TLS client config. Setting this option is not supported!

Login successful.

You have access to 241 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "openshift-operators".
Welcome! See 'oc help' to get started.
~/summit-2023-ossm-labs $ echo "[http://$](http://%24/)(oc get route istio-ingressgateway -o jsonpath='{.spec.host}' -n user25-dev-istio-system)"
[http://istio-ingressgateway-user25-dev-istio-system.apps.cluster-qnkf6.sandbox671.opentlc.com](http://istio-ingressgateway-user25-dev-istio-system.apps.cluster-qnkf6.sandbox671.opentlc.com/)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b2820214-d6b5-4aea-b80e-f599138ecca8/Untitled.png)

`echo "https://$(oc get route grafana -o jsonpath='{.spec.host}' -n user25-dev-istio-system)"`

### Task 2: Access the Observability Stack

In this task you will learn how you can find the URLs to the Observability Stack components of the Service Mesh.

- Logged in via CLI (we have used user `cristina`/`cristina`) execute the following to retrieve the URLs of the `OSSM` Observability Stack (`Kiali`, `Jaeger`, `Prometheus`, `Grafana`) components. Open the links on a browser and use the preceeding `cristina’s credentials to login.
    
    `./lab-3/login-as.sh cristina
    echo "http://$(oc get route kiali -o jsonpath='{.spec.host}' -n user25-dev-istio-system)"
    echo "https://$(oc get route jaeger -o jsonpath='{.spec.host}' -n user25-dev-istio-system)"
    echo "https://$(oc get route prometheus -o jsonpath='{.spec.host}' -n user25-dev-istio-system)"
    echo "https://$(oc get route grafana -o jsonpath='{.spec.host}' -n user25-dev-istio-system)"`
    
- An alternate method to access some of the components is once you have logged into `Kiali` you can access the `Grafana` and `Jaeger` URLs by clicking on **?** next to your name (top-right KIALI corner), then **About** and you should have the URLs presented.

### Task 3: Test the Observability stack as Product Owner for the Travel Demo Solution

Access the `Kiali` URL (see the CLI output) and login with username/password **`mus`**/**`mus`** (role: `Application Viewer`)

Follow the menu to the left to *Graph* and as the `Product Owner` you have **view** access to all 3 *data plane* namespaces and the *control plane* namespace.

![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m2/_images/02-mus-kiali-view.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m2/_images/02-mus-kiali-view.png)

You are allowed to:

1. See traces for the overall solution. From the `Kiali` menu on the left go to `Distributed Tracing` and login with your Openshift credentials (`mus/mus`) to view the tracing console (Authorize Access). Finally, select from the Service drop-down list `istio-ingressgateway`.
2. See metrics for the overall solution. Go to `Workloads` in `Kiali` and select `cars-v1` application workload. Use the `inbound` or `outbound` metrics.

![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m2/_images/02-mus-kiali-metrics.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m2/_images/02-mus-kiali-metrics.png)

- **Alternatively**, go to `Prometheus` (URL identified above) and login with your credentials (**`mus`**/**`mus`**). Apply on the `Graph` view ONE of the following metrics:
    - `istio_requests_total{destination_workload="discounts-v1", app="discounts"}` to visualize requests towards `discounts-v1`
    - `istio_request_duration_milliseconds_count{app="discounts"}`
    - `istio_response_bytes_bucket`
- See the dashboards in grafana for the solution. Access the `Grafana` URL log with credentials **`mus`**/**`mus`** (role: `Application Viewer`, See above on how to find the URL)
    - Check the 'status' of the overall Travel Agency solution **Dashboards → Manage → Istio → Istio Mesh Dashboard**

1. See the dashboards in grafana for the solution. Access the `Grafana` URL log with credentials **`mus`**/**`mus`** (role: `Application Viewer`, See above on how to find the URL)
    ◦ Check the 'status' of the overall Travel Agency solution **Dashboards → Manage → Istio → Istio Mesh Dashboard**
    ◦ Check the 'performance' of the overall Travel Agency solution **Dashboards → Manage → Istio → Istio Performance Dashboard**

![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m2/_images/02-grafana-istio-mesh-dashboard.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m2/_images/02-grafana-istio-mesh-dashboard.png)

![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m2/_images/02-grafana-performance.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m2/_images/02-grafana-performance.png)

**Verifying that RBAC restrictions for the `Product Owner` are in place**

As `Product Owner` You are not allowed to modify the Istio Configurations and view the Istio logs

- ou should not be able to modify configs via `Kiali` as user **`mus`**. If you select in the menu to the left `Istio Config`, select from the top drop-down namespace `user25-prod-istio-system`, filter by *Istio Type* `Gateway` and enter the `control-gateway` config you will notice the config cannot be modified by the user.
    
    ![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m2/_images/02-mus-view-config.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m2/_images/02-mus-view-config.png)
    
- In Kiali as **`mus`** you cannot access the logs of any workload, neither for `istio-proxy` nor for the main workload container. In the menu, to the left, go to `Workloads`, click on `cars-v1` workload and then go to tab `logs`, no logs are displayed. Instead, a similar message to the following is displayed:
    
    `Failed to fetch workload logs: pods "cars-v1-7b85d9b99b-l4hjg" 
    is forbidden: User "mus" cannot get resource "pods/log" in API group "" 
    in the namespace "userx-dev-travel-agency"`
    
    **Task 4 (Optional): Test the Observability stack as an App/Domain Owner for the Travel-Portal or Travel-Services Domain**
    
    Access `Kiali` URL with username/password **`farid`**/**`farid`** (role: `Mesh Developer`)
    
    - As the `Domain Owner` of the *Travel Services* domain you have **view** access to
        - *data plane* namespace `user25-dev-travel-agency` and the
        - *control plane* `user25-dev-istio-system` namespace.
        

# Setup Travel Demo `ServiceMeshControlPlane` for Production

## ABOUT THIS LAB

The Travel Agency company has a running development environment. Now 
the company is heading to production. In this scenario you are going to 
set up Openshift Service Mesh for production use, deploy the travel 
applications in the Mesh, exposing the application for external access 
with TLS and fine tune parameters for production.

## Production Requirements

Let us recall the initial requirements for production discussed in scenario 1:

1. The *Development Team* wants to be able to trace (every) request during development, and a sample of `20%` of traffic in Production
2. The *Development Teams* wants to implement some resiliency to make the overall application more stable and reliable.
3. The *Product Team* wants to see metrics around performance/usage (storing them for up to 1 week)
4. The *Security* team wishes to enable mTLS in all *intramesh* and *intermesh* communications
5. The *Platform Team* wants to centrally manage security

## User/Roles Mapping for the Production environment

For the purpose of managing, monitoring and troubleshooting the 
Production environment the following Enterprise Personas are involved.

| Username | Password | Enterprise Persona | Responsible for Namespace |
| --- | --- | --- | --- |
| emma | emma | Mesh Operator | user25-prod-istio-system |
| cristina | cristina | Travel Portal Domain Owner (Tech Lead) | user25-prod-travel-portal, user25-prod-travel-control |
| farid | farid | Travel Services Domain Owner (Tech Lead) | user25-prod-travel-agency |
| craig | craig | Platform (Application Ops) Team | user25-prod-travel-portal, user25-prod-travel-control, user25-prod-travel-agency |
| mus | mus | Product Owner | user25-prod-travel-portal, user25-prod-travel-control, user25-prod-travel-agency |

The Lab Instructors have already created all production namespaces for your user as well as the Openshift users that will interact with the mesh (including roles and rolebindings) for the production environment. You will now proceed to create the Prod Service Mesh Setup.

As this is a multi-tenant cluster you should restrict use for this lab to the following namespaces associated with your user **`user25-prod-istio-system`**, **`user25-prod-travel-control`**, **`user25-prod-travel-portal`**, **`user25-prod-travel-agency`**

Export the following in your [code-server](https://codeserver-codeserver-user25.apps.cluster-qnkf6.sandbox671.opentlc.com/) terminal and proceed to the location of the `ossm-labs` assets repository which you have cloned earlier.

## Task 1: Export Environment variables

|  | As this is a multi-tenant cluster you should restrict use for this lab to the following namespaces associated with your user user25-prod-istio-system, user25-prod-travel-control, user25-prod-travel-portal, user25-prod-travel-agency
Export the following in your https://codeserver-codeserver-user25.apps.cluster-qnkf6.sandbox671.opentlc.com/ terminal and proceed to the location of the ossm-labs assets repository which you have cloned earlier.
• export CLUSTER_API=https://api.cluster-qnkf6.sandbox671.opentlc.com:6443
• export LAB_PARTICIPANT_ID=user25
• export OCP_DOMAIN=apps.cluster-qnkf6.sandbox671.opentlc.com
• export SSO_CLIENT_SECRET=bcd06d5bdd1dbaaf81853d10a66aeb989a38dd51 |
| --- | --- |

If you are running out of time and wish to complete the following lab sections in a single step execute

`cd lab-3
./complete-lab-3.sh apps.cluster-qnkf6.sandbox671.opentlc.com user25`

## 

## Task 2: Install a basic Service Mesh Control Plane with an external Jaeger configuration

You are going to create a basic Service Mesh Control Plane for production. Regarding the Tracing configuration, `Red Hat Openshift Service Mesh (OSSM)` makes the following 2 suggestions on setting up tracing for the production environment. `Option 2`, the *fully customized* option, has been selected for the production setup.

- Option 1: [Production distributed tracing platform deployment (minimal) - via SMCP Resource](https://docs.openshift.com/container-platform/4.12/service_mesh/v2x/ossm-deploy-production.html#ossm-smcp-prod_ossm-architecture)
- Option 2: [Production distributed tracing platform deployment (fully customized)](https://docs.openshift.com/container-platform/4.12/service_mesh/v2x/ossm-reference-jaeger.html#ossm-deploying-jaeger-production_jaeger-config-reference)

Login as Mesh Operator (credentials: `emma/emma`) and run the `create-prod-smcp-1-tracing.sh` script (follow its output). This will deploy the production `SMCP` resource **`user25-production`** and an external fully customized `Jaeger` instance in your lab user’s Service Mesh controlplane namespace **`user25-prod-istio-system`**.

`cd lab-3
./login-as.sh emma
./create-prod-smcp-1-tracing.sh user25-prod-istio-system user25-production user25-jaeger-small-production`

- The `Jaeger Operator` will also create a `Jaeger Collector`, a `Jaeger Query` and an `Elastic Search` Deployment in your `user25-prod-istio-system` in order to collect and backup the traces.
    
    This is the Jaeger Custom Resource applied:
    
    `kind: Jaeger
    metadata:
      name:  user25-jaeger-small-production
    spec:
      strategy: production 
      storage:
        type: elasticsearch 
        esIndexCleaner:
          enabled: true
          numberOfDays: 7 
          schedule: '55 23 * * *'
        elasticsearch:
          nodeCount: 1 
          storage:
            size: 1Gi 
          resources:  
            requests:
              cpu: 200m
              memory: 1Gi
            limits:
              memory: 1Gi
          redundancyPolicy: ZeroRedundancy`
    
    The applied `Jaeger` setup will ensure that:
    
    - **(1)** Production focused setup is applied
    - **(2)** Backed up for persistence by Elastic Search
    - **(3)** With indexes deleted every 7 days
    - **(4)** Elastic Search will be hosted on a single Elastic node
    - **(5)** Total Elastic Search Index size will be *`1Gi`*
    - **(6)** Resource for the node will be both requested and limited
    - **(7)** Since a single node is setup redundancy of the indices will be set to `ZeroRedundancy`
- This is the SMCP Resource that is configured to use the external Jaeger instance:
    
    `apiVersion: maistra.io/v2
    kind: ServiceMeshControlPlane
    metadata:
      name: production
    spec:
      security:
        dataPlane:
          automtls: true
          mtls: true
      tracing:
        sampling: 2000 
        type: Jaeger
      general:
        logging:
          logAsJSON: true
      profiles:
        - default
      proxy:
        accessLogging:
          file:
            name: /dev/stdout
        networking:
          trafficControl:
            inbound: {}
            outbound:
              policy: REGISTRY_ONLY 
      policy:
        type: Istiod
      addons:
        grafana:
          enabled: true
        jaeger:  
          install:
            ingress:
              enabled: true
            storage:
              type: Elasticsearch 
          name:  user25-jaeger-small-production 
        kiali:
          enabled: true
        prometheus:
          enabled: true
      version: v2.2
      telemetry:
        type: Istiod"`
    
    The applied `ServiceMeshControlPlane` Resource ensures that:
    
    - **(1)** 20% of all traces (as requested by the developers) will be collected,
    - **(2)** No external outgoing communications to a host not registered in the mesh will be allowed,
    - **(3)** `Jaeger` resource will be available in the `Service Mesh` for traces storage,
    - **(4)** It will utilize Elastic Search for persistence of traces (unlike in the `dev-istio-system` namespace where `memory` is utilized)
    - **(5)** The ` user25-jaeger-small-production` external `Jaeger` Resource is integrated by and utilized in the `Service Mesh`.

Login to the Openshift console with Mesh Operator credentials `emma/emma` and navigate to **`Administrator`** → **`Workloads`** → **`Pods`**  in namespace `user25-prod-istio-system` namespace. Verify all deployments and pods are running.

![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m3/_images/03-prod-istio-system.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m3/_images/03-prod-istio-system.png)

|  | The configs came from https://github.com/skoussou/summit-2023-ossm-labs/blob/main/lab-3/create-prod-smcp-1-tracing.sh script which you can inspect for details. |
| --- | --- |

## Task 3: Add the Application Namespaces to the Production Mesh and create the Deployments

In this task you will add the application namespaces to our newly created Service Mesh by specifying `ServiceMeshMember`
 resources and deploying the corresponding applications for production. 
You will also configure the applications for the usage within the 
Service Mesh by specifying two `sidecar` containers:

1. `istio-proxy` sidecar container: used to proxy all communications in/out of the main application container and apply `Service Mesh` configurations
2. `jaeger-agent` sidecar container: The `Service Mesh` documentation [Jaeger Agent Deployment Best Practices](https://docs.openshift.com/container-platform/4.11/service_mesh/v2x/ossm-reference-jaeger.html#distr-tracing-deployment-best-practices_jaeger-config-reference) mentions the options of deploying `jaeger-agent` as sidecar or as `DaemonSet`. In order to allow `multi-tenancy` in this Openshift cluster the former has been selected.

All application `Deployment`(s) will be patched as follows to include the sidecars (**Warning:** Don’t apply as the script `deploy-travel-services-domain.sh` further down will do so):

`oc patch deployment/voyages -p '{"metadata":{"annotations":{"sidecar.jaegertracing.io/inject": " user25-jaeger-small-production"}}}' -n $ENV-travel-portal
oc patch deployment/voyages -p '{"spec":{"template":{"metadata":{"annotations":{"sidecar.istio.io/inject": "true"}}}}}' -n $ENV-travel-portal`

Now let’s get started.

- Login as Mesh Developer (credentials `farid/farid`) who is responsible for the Travel Agency services and check the Labels for the `user25-prod-travel-agency` application namespace
    
    `./login-as.sh farid
    ./check-project-labels.sh user25-prod-travel-agency`
    
    The result of this command should look similar to this:
    
    `{
      "kubernetes.io/metadata.name": "user25-prod-travel-agency"
    }`
    
- Next add the application namespaces to the Production Service Mesh Tenant and check the Labels again
    
    `./create-membership.sh user25-prod-istio-system user25-production user25-prod-travel-agency
    
    ./check-project-labels.sh user25-prod-travel-agency`
    
    The result of this command should look similar to this (you may need to retry a few times until all labels are applied):
    
    `{
      "kiali.io/member-of": "user25-prod-istio-system",
      "kubernetes.io/metadata.name": "user25-prod-travel-agency",
      "maistra.io/member-of": "user25-prod-istio-system"
    }`
    
- Next you will deploy the Travel Agency Services applications and inject the sidecar containers.
    
    `./deploy-travel-services-domain.sh prod prod-istio-system user25`
    
    You can also login as `farid/farid` in the Openshift Console and verify the application PODs have started in your `user25-prod-travel-agency` namespace (navigate to **`Administrator`** → **`Workloads`** → **`Pods`**). It should look like:
    
    ![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m3/_images/03-travel-agency-expected-3-container-pods.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m3/_images/03-travel-agency-expected-3-container-pods.png)
    
- In the next step you will install the second set of applications, the Travel Control and Travel Portal apps, with the responsible user `cristina/cristina`
    
    `./login-as.sh cristina
    ./check-project-labels.sh user25-prod-travel-control
    ./check-project-labels.sh user25-prod-travel-portal`
    
- Add the `user25-prod-travel-control` application namespace to the Mesh
    
    `./create-membership.sh user25-prod-istio-system user25-production user25-prod-travel-control
    
    ./check-project-labels.sh user25-prod-travel-control`
    
- Add the `user25-prod-travel-portal` application namespace to the Mesh
    
    `./create-membership.sh user25-prod-istio-system user25-production user25-prod-travel-portal
    
    ./check-project-labels.sh user25-prod-travel-portal`
    
- Next you will deploy the Travel Portal and Travel Control applications and inject the sidecars.
    
    `./deploy-travel-portal-domain.sh prod prod-istio-system apps.cluster-qnkf6.sandbox671.opentlc.com user25`
    
- Login with `cristina/cristina` in the Openshift Console and verify that the applications have been created and are running in the two namespaces:
    - `user25-prod-travel-control`
        
        ![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m3/_images/03-travel-control-expected-3-container-pods.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m3/_images/03-travel-control-expected-3-container-pods.png)
        
    - `user25-prod-travel-portal`
        
        ![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m3/_images/03-travel-portal-expected-3-container-pods.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m3/_images/03-travel-portal-expected-3-container-pods.png)
        

## Task 4: Expose the Travel Portal Dashboard via TLS

After the deployment of the applications you will make them 
accessible outside of the cluster for the Travel Agency customers 
exposing the services with a custom TLS cert.
In order to achieve that,

- you are going to create a TLS certificate
- store it in a secret in our SMCP namespace
- create on Openshift passthrough route forwarding traffic to the Istio ingress Gateway
- create an Istio Gateway Resource configured with our TLS certificate

Right now if you login to the **production** [Kiali Dashboard](https://kiali-user25-prod-istio-system.apps.cluster-qnkf6.sandbox671.opentlc.com/) with the user `emma/emma` (**Istio Config** → filter by `VirtualService`) , there is an issue in the `VirtualService` resource `control` and an error on Kiali as no `Gateway` exists yet.

![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m3/_images/03-no-gw-for-travel-control-ui-vs.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m3/_images/03-no-gw-for-travel-control-ui-vs.png)

Login as Mesh Operator (credentials `emma/emma`) and execute the following script (follow the output) to achieve the above.

`./login-as.sh emma
./create-https-ingress-gateway.sh prod-istio-system apps.cluster-qnkf6.sandbox671.opentlc.com user25`

|  | The configs come from https://github.com/skoussou/summit-2023-ossm-labs/blob/main/lab-3/create-https-ingress-gateway.sh script which you can inspect for details. |
| --- | --- |

After finishing, the script above, you’ll get the exposed URL Route and the `Travel Control Dashboard` should be accessible at [https://travel-user25.apps.cluster-qnkf6.sandbox671.opentlc.com](https://travel-user25.apps.cluster-qnkf6.sandbox671.opentlc.com/) and the `Kiali` error on the `VirtualService` resource `control` should now have been resolved.

![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m3/_images/03-Travel-Control-Dashboard-https.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m3/_images/03-Travel-Control-Dashboard-https.png)

## Task 5: Configure Prometheus for Production

In order to configure Prometheus for production there are several options:

Option 1: Create a `PersistenceVolume` for the `SMCP` created `Prometheus` resourceWith this option the `mesh operator` will enhance the `SMCP` managed `Prometheus Deployment` resource in order to
• extend metric retention to 7 days (`7d`) and
• enable long-term persistence of the metrics by adding a persistent volume to the deployment.Option 2: External `Prometheus` Setup via `prometheus-operator`With this option the `cluster admin` user will perform the following actions:
1. Deploy an additional `Prometheus Operator` in `prod-istio-system`
2. Deploy a `StatefulSet` based `Prometheus` resource with 2 replicas
3. Configure the prometheus replicas to monitor the components in `prod-istio-system` and all dataplane namespaces.Option 3: Integrate with Openshift `Monitoring` StackWith this option only the `dataplane` metrics (`istio-proxy`
 and business container) are collected. These will be scraped by the 
Openshift Monitoring Stack’s Prometheus and the changes required on the 
service mesh are described in [How to configure user-workload to monitor ServiceMesh application in Openshift 4](https://access.redhat.com/solutions/6958679).Option 4: Integrate with an external `Monitoring` ToolThis option assumes that another tool like Datadog is used by the Operations team to collect metrics. In order to achieve this:
1. For `controlplane` components metrics collection, the tool needs to be part of the control plane namespace or a `NetworkPolicy` to allow it visibility to those components is required.g
2. For `dataplane` metrics the same approach described, previously, in *Option 3* is to be followed.

For the purpose of this lab you will deliver **Option 1** in the production setup. Login as `Mesh Operator` (credentials `emma/emma`), the script below will help you create a `PVC` for Prometheus and update the Prometheus configuration to utilize it and extend metrics retention to `168h`.

`./login-as.sh emma
./update-prod-smcp-2-prometheus.sh user25-prod-istio-system`

|  | The configs come from https://github.com/skoussou/summit-2023-ossm-labs/blob/main/lab-3/update-prod-smcp-2-prometheus.sh script which you can inspect for details. |
| --- | --- |

## Task 6: Final Production Configuration

The following **Purpose** and **Principals** have been finalized with the `Travel Agency` architects and final `Service Mesh` configuration tunings have been accepted based on these:

- **Purpose:**
    - Secure service-to-service communications.
    - Monitor usage and health of the inter-service communications.
    - Allow separate teams to work in isolation whilst delivering parts of a solution.
- **Principals:**
    - An external mechanism of configuration of traffic encryption, authentication and authorization.
    - Transparent integration of additional services of expanding functionality.
    - An external traffic management and orchestration mechanism.
    - All components will be configured with High Availability in mind.
    - Observability is to be used for verification of system "sound operation", not auditing.

Therefore, based on these purpose and principals the final `PROD` setup will apply the following:

- *Tracing:* used only for debug purposes (rather than as sensitive -auditing- information), a sample **5%** of all traces will only be collected, whilst these are going to be stored for **7 Days**. Elastic Search cluster will be used for this long-term storage.
- *Metrics:* will have long-term storage (**7 Days**) with further archiving of the metrics beyond this period in order to assist historical comparisons
- *Grafana:* will have persistance storage
- *Istio Ingress/Egress Gateways:* (scale up to 2 instances)
- *Istiod Controlplane* (scale up to 2 instances)

To apply the final production `SMCP` tuning, login as Mesh operator (credentials `emma/emma`)
 and execute the final update script. Follow the script logs to 
understand the changes applied. On a separate terminal you can execute `oc get pods -w -n user25-prod-istio-system` to follow the POD scalings.

`./login-as.sh emma
./update-prod-smcp-3-final.sh user25-prod-istio-system user25-production user25-jaeger-small-production`

|  | The configs come from https://github.com/skoussou/summit-2023-ossm-labs/blob/main/lab-3/update-prod-smcp-3-final.sh script which you can inspect for details. |
| --- | --- |

# Securing (Authn & Authz) Gateways with MTLS and valid JWT Token

## Task 1: Export Environment variables

|  | As this is a multi-tenant cluster you should restrict use for this lab to the following namespaces associated with your user user25-prod-istio-system, user25-prod-travel-control, user25-prod-travel-portal, user25-prod-travel-agency
Export the following in a terminal in your https://codeserver-codeserver-user25.apps.cluster-qnkf6.sandbox671.opentlc.com/.
• export CLUSTER_API=https://api.cluster-qnkf6.sandbox671.opentlc.com:6443
• export LAB_PARTICIPANT_ID=user25
• export OCP_DOMAIN=apps.cluster-qnkf6.sandbox671.opentlc.com
• export SSO_CLIENT_SECRET=bcd06d5bdd1dbaaf81853d10a66aeb989a38dd51 |
| --- | --- |

|  | If you are running out of time and wish to complete the following lab sections in a single step execute

cd lab-4
./complete-lab-4.sh $SSO_CLIENT_SECRET apps.cluster-qnkf6.sandbox671.opentlc.com user25 |
| --- | --- |

## Task 2: External API integration with mTLS

In this task you are going to create:

- An additional ingress gateway for the Travel Agency API Services
- An Istio `VirtualService` configuration for traffic to flow via the *gto* gateway
- Certificates for our `curl` GTO client to connect with MTLS to the *gto* gateway
- Verify the MTLS connection

### Step 1 - Create additionalIngress Gateway in SMCP

Login as Mesh Operator (credentials `emma/emma`) and modify the SMCP resource. You can also do this from within the Openshift Console.

`cd lab-4
./login-as.sh emma
oc edit smcp/user25-production -n user25-prod-istio-system`

Navigate to the `gateways` section and add an additional gateway as follows:

  `gateways:
    additionalIngress:
      gto-user25-ingressgateway:
        enabled: true
        runtime:
          deployment:
            autoScaling:
              enabled: false
        service:
          metadata:
            labels:
              app: gto-user25-ingressgateway
          selector:
            app: gto-user25-ingressgateway`

You can verify the creation of the additional gateway either in the OCP Console or with the CLI:

`$ oc get pods -n user25-prod-istio-system |grep gto
gto-user25-ingressgateway-5c87989fb7-r9grv

$ oc get routes -n user25-prod-istio-system |grep "ingress"
gto-user25-ingressgateway   gto-user25-ingressgateway-user25-prod-istio-system.apps.cluster-xvsnq.sandbox2004.opentlc.com          gto-user25-ingressgateway        8080                               None`

### Step 2 - Expose and secure via MTLS host on new Gateway

In the next step you will expose the new `Gateway` and in order to do this the script provided will:

- create the CA and certs for the exposure of the TLS based `Gateway`,
- an Openshift passthrough route,
- the Istio `Gateway` configuration
- create the client certificates based on the same CA for the curl client (in order to test MTLS):

`./create-external-mtls-https-ingress-gateway.sh prod-istio-system apps.cluster-qnkf6.sandbox671.opentlc.com user25`

You can check the created certs by looking in your current directory:

`ls -ltr

-rw-r--r--@ 1 oschneid  staff  3272 Dec 19 11:04 ca-root.key
-rw-r--r--@ 1 oschneid  staff  1944 Dec 19 11:04 ca-root.crt
-rw-r--r--@ 1 oschneid  staff   523 Dec 19 11:04 gto-user25.conf
-rw-r--r--@ 1 oschneid  staff  1704 Dec 19 11:04 gto-user25-app.key
-rw-r--r--@ 1 oschneid  staff  1045 Dec 19 11:04 gto-user25-app.csr
-rw-r--r--@ 1 oschneid  staff    17 Dec 19 11:04 ca-root.srl
-rw-r--r--@ 1 oschneid  staff  1614 Dec 19 11:04 gto-user25-app.crt
-rw-r--r--@ 1 oschneid  staff  1704 Dec 19 11:04 curl-client.key
-rw-r--r--@ 1 oschneid  staff   940 Dec 19 11:04 curl-client.csr
-rw-r--r--@ 1 oschneid  staff  1497 Dec 19 11:04 curl-client.crt`

To verify what has been applied you can navigate in Kiali to `Istio Config` and check the `travel-api-gateway` `Gateway` resource. The resource will now:

- Expose host `gto-user25.apps.cluster-qnkf6.sandbox671.opentlc.com` over https
- TLS mode for this host will be `MUTUAL`
- Certificates will be used from `Secret` resource `gto-user25-secret`

![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m4/_images/04-Kiali-Gateway.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m4/_images/04-Kiali-Gateway.png)

|  | The configs came from https://github.com/skoussou/summit-2023-ossm-labs/blob/main/lab-4/create-external-mtls-https-ingress-gateway.sh script which you can inspect for details. |
| --- | --- |

### Step 3 - Configuration to allow Traffic flow via new Gateway

As the Travel Services Domain Owner (Tech Lead) you can now enable 
Istio routing to your services via the new gateway (previously only 
possible via `user25-prod-travel-portal` namespace). Login with credentials `farid/farid` and deploy the Istio Configs in your `user25-prod-travel-agency`
 namespace to allow requests via the above defined Gateway to reach the 
required services cars, insurances, flights, hotels and travels.

`./login-as.sh farid
./deploy-external-travel-api-mtls-vs.sh user25-prod user25-prod-istio-system user25`

The script will also run some example requests and if MTLS handshake works you should see something similar to this:

![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m4/_images/04-MTLS-reqs.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m4/_images/04-MTLS-reqs.png)

You can now go to the Kiali Dashboard (Graph section) and observe the
 traffic entering the Mesh through the MTLS enabled Gateway.

![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m4/_images/04-gto-external-ingressgateway.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m4/_images/04-gto-external-ingressgateway.png)

|  | The configs came from https://github.com/skoussou/summit-2023-ossm-labs/blob/main/lab-4/deploy-external-travel-api-mtls-vs.sh script which you can inspect for details. |
| --- | --- |

## Task 3: Configure Authn and Authz with JWT Tokens

The Travel Agency has exposed their API services with MTLS through an
 additional ingress gateway. Now they want to further lock down who 
should be able to access their services. Therefore they want to use JWT 
Tokens with Istio.

|  | The Lab Instructors have created an RH-SSO Identity Provider, a Realm for Service Mesh and have also created a client configuration (istio-user25-production) for your user25-production Service Mesh control plane. You will now use this setup. |
| --- | --- |

### The JWT workflow

The intended final authentication workflow (in addition to the mTLS handshake) for external requests with a `JWT` token is as follows:

1. The external user authenticates to RHSSO and gets a JWT token
2. The user performs a HTTP request to `[/travels](https://gto-user25.apps.cluster-qnkf6.sandbox671.opentlc.com/travels/Brussels)` (or one of `cars`, `hotels`, `insurances`, `flights`) and passes along this request the JWT token
3. The `istio-proxy` container of the Istio Ingress Gateway checks the validity of the JWT token based on the `RequestAuthentication` and `AuthorizationPolicy` objects
4. If the JWT token is valid and the `AuthorizationPolicy` matches, the external user is allowed to access the `/PATH` - otherwise, an error message is returned to the user (code `403`, message `RBAC: access denied` or others).
    ◦ Pros:
        ▪ This is the simplest approach (only 2 Custom Resources to be deployed)
        ▪ Fine-grained authorization based on JWT token fields
    ◦ Cons:
        ▪ No OIDC workflow: The user must get a JWT token on its own, and pass it with the HTTP request on its own
        ▪ Need to define `RequestAuthentication` and `AuthorizationPolicy` objects for each application inside the service mesh

### Step 1 - Define Authentication and Authorization with valid RHSSO JWT Token

As the communications between RHSSO and `istiod` are secured with a router certificate the `Mesh Operator` has to perform a one-time operation first to load the certificate to `istiod`. This is performed by the following script:

`./login-as.sh emma
./mount-rhsso-cert-to-istiod.sh user25-prod-istio-system user25-production apps.cluster-qnkf6.sandbox671.opentlc.com`

The `RequestAuthentication` enables JWT validation on the Istio ingress gateway so that the validated JWT claims can later be used (i.e. in a `VirtualService`) for routing purposes. The request authentication is applied on the ingress gateway because the JWT claim based routing is **only** supported on ingress gateways.

As the current Travel Agency decision is to have a producer/consumer 
model for their Service Mesh these changes are performed as Mesh 
Operator (`emma/emma`) in the controlplane namespace based gateway `gto-user25-ingressgateway`.

|  | The RequestAuthentication will only check the JWT if it 
exists in the request. To make the JWT required and reject the request 
if it does not include JWT, apply an authorization policy. |
| --- | --- |

`./login-as.sh emma

echo "apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
 name: jwt-rhsso-gto-external
 namespace: user25-prod-istio-system
spec:
 selector:
   matchLabels:
     app: gto-user25-ingressgateway
 jwtRules:
   - issuer: >-
       https://keycloak-rhsso.apps.cluster-qnkf6.sandbox671.opentlc.com/auth/realms/servicemesh-lab
     jwksUri: >-
       https://keycloak-rhsso.apps.cluster-qnkf6.sandbox671.opentlc.com/auth/realms/servicemesh-lab/protocol/openid-connect/certs" | oc apply -f -`

Next add an `AuthorizationPolicy` Resource which specifies to only allow requests from a user when the token was issued by the specified RH-SSO.

`echo "apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: authpolicy-gto-external
  namespace: user25-prod-istio-system
spec:
  selector:
    matchLabels:
      app: gto-user25-ingressgateway
  action: ALLOW
  rules:
  - from:
    - source:
        requestPrincipals: ['*']
    when:
    - key: request.auth.claims[iss]
      values: ['https://keycloak-rhsso.apps.cluster-qnkf6.sandbox671.opentlc.com/auth/realms/servicemesh-lab'] " | oc apply -f -`

## Task 4: Test Authn / Authz with JWT

- You are ready to test if the external access is secured by sending a request to the */cars* and */travels* APIs without a JWT Token. The following should now result in a `HTTP 403` Response (RBAC / Access Denied):
    
    `./login-as.sh emma
    
    export GATEWAY_URL=$(oc -n user25-prod-istio-system get route gto-user25 -o jsonpath='{.spec.host}')
    echo $GATEWAY_URL
    echo "------------------------------------------------------------"
    curl -v --cacert ca-root.crt --key curl-client.key --cert curl-client.crt https://$GATEWAY_URL/cars/Tallinn
    echo
    echo "------------------------------------------------------------"
    curl -v --cacert ca-root.crt --key curl-client.key --cert curl-client.crt https://$GATEWAY_URL/travels/Tallinn
    echo`
    
- Next, Authenticate against the RH-SSO instance and retrieve a JWT Access Token:
    
    `TOKEN=$(curl -Lk --data "username=gtouser&password=gtouser&grant_type=password&client_id=istio-user25&client_secret=$SSO_CLIENT_SECRET" https://keycloak-rhsso.apps.cluster-qnkf6.sandbox671.opentlc.com/auth/realms/servicemesh-lab/protocol/openid-connect/token | jq .access_token)
    
    echo $TOKEN`
    
- Now you can start sending requests with the JWT Token to the additional Ingress Gateway by using MTLS:
    
    `./call-via-mtls-and-jwt-travel-agency-api.sh user25-prod-istio-system gto-user25 $TOKEN`
    

Login to Kiali, go to menu `Graph`, select only namespace `user25-prod-istio-system` and verify the traffic is successfully entering the mesh.

![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m4/_images/04-gto-external-ingressgateway-jtw.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m4/_images/04-gto-external-ingressgateway-jtw.png)

[Securing (Authn & Authz) Gateways with MTLS and valid JWT Token](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m4/intro.html?LAB_PARTICIPANT_ID=user25&OCP_DOMAIN=apps.cluster-qnkf6.sandbox671.opentlc.com&API_URL=https%3A%2F%2Fapi.cluster-qnkf6.sandbox671.opentlc.com%3A6443)

# Part I: Apply Business Authorization Restrictions

## Task 1: Export Environment variables

|  | As this is a multi-tenant cluster you should restrict use for this lab to the following namespaces associated with your user user25-prod-istio-system, user25-prod-travel-control, user25-prod-travel-portal, user25-prod-travel-agency
Export the following in a terminal in your https://codeserver-codeserver-user25.apps.cluster-qnkf6.sandbox671.opentlc.com/.
• export CLUSTER_API=https://api.cluster-qnkf6.sandbox671.opentlc.com:6443
• export LAB_PARTICIPANT_ID=user25
• export OCP_DOMAIN=apps.cluster-qnkf6.sandbox671.opentlc.com
• export SSO_CLIENT_SECRET=bcd06d5bdd1dbaaf81853d10a66aeb989a38dd51 |
| --- | --- |

|  | If you are running out of time and wish to complete the following lab sections in a single step execute

cd lab-5
./complete-lab-5.sh $SSO_CLIENT_SECRET apps.cluster-qnkf6.sandbox671.opentlc.com user25 |
| --- | --- |

## Task 2: Applying default authorization policies

The Travel Agency company, like any other business, requires fine-grained *authorization*
 policies for its systems. Openshift Service Mesh provides the 
capability to externalize these policies from the actual service code 
and the *Travel Agency* `Mesh Operator` will implement them restricting access based on known *Best Practices* and business requirements.

Further authorization capabilities are described in the `Istio` [authorization documentation](https://istio.io/latest/docs/tasks/security/authorization/).

### Step 1 - Verify current default AUTHZ is `ALLOW` all

The *Service Mesh* default Authz policy is `ALLOW` all.

First lets verify that by default the *Service Mesh* authorization policies allows all communications. The following table determines the expected allowed communications.

| Type of Policy | Namespaces | Client | Target |
| --- | --- | --- | --- |
| ALLOW all | prod-istio-system → prod-travel-control | Browser | control.prod-travel-control |
| ALLOW all | prod-istio-system → prod-travel-agency | gto-user25-ingressgateway | travels.prod-travel-agency, flights.prod-travel-agency, hotels.prod-travel-agency, insurances.prod-travel-agency, cars.prod-travel-agency |
| ALLOW all | prod-travel-control → prod-travel-agency | control.prod-travel-control | travels.prod-travel-agency, flights.prod-travel-agency, hotels.prod-travel-agency, insurances.prod-travel-agency, cars.prod-travel-agency |
| ALLOW all | prod-travel-portal → prod-travel-agency | viaggi.prod-travel-portal | travels.prod-travel-agency, flights.prod-travel-agency, hotels.prod-travel-agency, insurances.prod-travel-agency, cars.prod-travel-agency |
| ALLOW all | prod-travel-agency → prod-travel-agency | travels.prod-travel-agency | travels.prod-travel-agency, flights.prod-travel-agency, hotels.prod-travel-agency, insurances.prod-travel-agency, cars.prod-travel-agency |

Verify the default communication paths described in the table above. Login as Mesh Operator (credentials `emma/emma`) and execute the following script:

`cd lab-5
./login-as.sh emma

./check-authz-all.sh ALLOW user25-prod-istio-system apps.cluster-qnkf6.sandbox671.opentlc.com $SSO_CLIENT_SECRET user25`

### Step 2 - Apply best practice security pattern to `DENY` all

In the previous lab [Step 1 - Define Authentication and Authorization with valid RHSSO JWT Token](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m4/walkthrough.html?LAB_PARTICIPANT_ID=user25&OCP_DOMAIN=apps.cluster-qnkf6.sandbox671.opentlc.com&API_URL=https%3A%2F%2Fapi.cluster-qnkf6.sandbox671.opentlc.com%3A6443#_step_1__define_authentication_and_authorization_with_valid_rhsso_jwt_token) you applied an `AuthorizationPolicy` resource which allowed requests via the `gto-user25-ingressgateway`. Now, you will utilize the `default-deny` pattern to DENY requests unless there is a specific `AuthorizationPolicy` allowing it.

As Mesh Operator with `emma/emma`  apply the `default-deny` pattern to the prod-travel-agency namespace

`echo "apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-nothing
  namespace: user25-prod-travel-agency
spec:
  {}" | oc apply -f -`

and the prod-travel-control namespace:

`echo "apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-nothing
  namespace: user25-prod-travel-control
spec:
  {}  " | oc apply -f -`

### Step 3 - Verify `DENY` all is applied

Now you can verify that no communications from the *Service Mesh* are authorizated towards the *Travel Agency API* Services or the *Travel Portal*.

| Type of Policy | Namespaces | Client | Target |
| --- | --- | --- | --- |
| DENY all | prod-istio-system → prod-travel-control | Browser | https://travel-user25.apps.cluster-qnkf6.sandbox671.opentlc.com/ |
| DENY all | prod-istio-system → prod-travel-agency | gto-user25-ingressgateway | travels.prod-travel-agency, flights.prod-travel-agency, hotels.prod-travel-agency, insurances.prod-travel-agency, cars.prod-travel-agency |
| DENY all | prod-travel-control → prod-travel-agency | control.prod-travel-control | travels.prod-travel-agency, flights.prod-travel-agency, hotels.prod-travel-agency, insurances.prod-travel-agency, cars.prod-travel-agency |
| DENY all | prod-travel-portal → prod-travel-agency | viaggi.prod-travel-portal | travels.prod-travel-agency, flights.prod-travel-agency, hotels.prod-travel-agency, insurances.prod-travel-agency, cars.prod-travel-agency |
| DENY all | prod-travel-agency → prod-travel-agency | travels.prod-travel-agency | travels.prod-travel-agency, flights.prod-travel-agency, hotels.prod-travel-agency, insurances.prod-travel-agency, cars.prod-travel-agency |

Let us check the communication paths again:

`./check-authz-all.sh DENY user25-prod-istio-system apps.cluster-qnkf6.sandbox671.opentlc.com $SSO_CLIENT_SECRET user25`

You can also login to Kiali and verify the traffic in the Dashboard:

![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m5/_images/05-DENY-ALL-KIALI.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m5/_images/05-DENY-ALL-KIALI.png)

### Step 4 - Authz policy to allow Travel Dashboard UI access

After applying the DENY ALL policies, authorize access only to the required paths to make the applications work again.

Let us first login as Mesh Operator with `emma/emma` and check if you can access the Travel Dashboard. This should return a RBAC Access Denied error.

`./login-as.sh emma

curl -k https://travel-user25.apps.cluster-qnkf6.sandbox671.opentlc.com/

RBAC: access denied`

Now create the following AuthorizationPolicies:

`echo "apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: authpolicy-istio-ingressgateway
  namespace: user25-prod-istio-system
spec:
  selector:
    matchLabels:
      app: istio-ingressgateway
  rules:
    - to:
        - operation:
            paths: [\"*\"]" |oc apply -f -`

and

`echo "apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-selective-principals-travel-control
  namespace: user25-prod-travel-control
spec:
  action: ALLOW
  rules:
    - from:
        - source:
            principals: [\"cluster.local/ns/user25-prod-istio-system/sa/istio-ingressgateway-service-account\"]"|oc apply -f -`

Please verify the access to the Travel Dashboard again. It should be 
accessible right now. You can also open the URL in your Browser:

`curl -k https://travel-user25.apps.cluster-qnkf6.sandbox671.opentlc.com/`

### Step 5 - Apply fine grained business Authz policies for service to service communications

In this last step, you will create authorisation policies which will allow access:

- from `gto-user25` gateway towards
    - `travels.user25-prod-travel-agency`,
    - `hotels.user25-prod-travel-agency`,
    - `cars.user25-prod-travel-agency`,
    - `insurances.user25-prod-travel-agency`,
    - `flights.user25-prod-travel-agency` in order to enable external partner requests
- for intra `user25-prod-travel-agency` communications
- from `user25-prod-travel-portal` to `user25-prod-travel-agency`

Login as Mesh Developer with `farid/farid` and create the following AuthorizationPolicy:

`./login-as.sh farid

echo "apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
 name: allow-selective-principals-travel-agency
 namespace: user25-prod-travel-agency
spec:
 action: ALLOW
 rules:
   - from:
       - source:
           principals: [\"cluster.local/ns/user25-prod-istio-system/sa/gto-user25-ingressgateway-service-account\",\"cluster.local/ns/user25-prod-travel-agency/sa/default\",\"cluster.local/ns/user25-prod-travel-portal/sa/default\"]" |oc apply -f -`

Verify all communications meet the fine-grained authorization targets set by the Travel Agency

`./login-as.sh emma

./check-authz-all.sh 'ALLOW intra' user25-prod-istio-system apps.cluster-qnkf6.sandbox671.opentlc.com $SSO_CLIENT_SECRET user25`

Please also login to Kiali and observe the communication flows:

![https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m5/_images/05-access-restored-with-authz-policies.png](https://guides-guides.apps.cluster-qnkf6.sandbox671.opentlc.com/summit-ossm-labs-guides/main/m5/_images/05-access-restored-with-authz-policies.png)

## Task 3(Optional): Disable STRICT MTLS for specific services

The Service Mesh of the Travel Agency company is configured to automatically use mTLS:

Excerpt from the SMCP

`spec:
  security:
    dataPlane:
      automtls: true
      mtls: true`

but sometimes there is the requirement to exclude specific services from `OSSM` **mTLS**, i.e. if workloads offer their own mTLS certificates (see KAFKA, Elastic Search).

In addition if the SMCP configuration doesn’t actually enforce mTLS, this can be done by configuring a `PeerAuthentication` resource.

|  | A PeerAuthentication resource defines how traffic will be tunneled (or not) to the sidecar proxy. |
| --- | --- |

Although, it is not necessary for our use case to do so if at the end of the lab there is still time left you can try to `DISABLE`/`RE-ENABLE` the MTLS setting in the mesh for the `cars` service by following the instruction below in order to become familiar with this capability.

### Step 1 - Verify Production `ServiceMeshControlPlane` strict MTLS setting

First login as Mesh Developer with `farid/farid` and check the global mTLS configurations in the control plane namespace:

`cd lab-5

./login-as.sh farid

oc get peerauthentication -n user25-prod-istio-system

NAME                            MODE         AGE
default                         STRICT       4d1h
disable-mtls-jaeger-collector   DISABLE      4d1h
grafana-ports-mtls-disabled     PERMISSIVE   4d1h`

### Step 2 - How to disable strict MTLS for a service?

Then disable strict *MTLS* for the cars service by applying a PeerAuthentication resource in the applications namespace:

`echo "apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: cars-mtls-disable
  namespace: user25-prod-travel-agency
spec:
  selector:
    matchLabels:
      app: cars
  mtls:
    mode: DISABLE"|oc apply -f -`

Check the applied resource

`oc get peerauthentication -n user25-prod-travel-agency

NAME                MODE      AGE
cars-mtls-disable   DISABLE   47s`

### Step 3 - Validate no MTLS activity

Validate no mTLS handshaking is taking place, by connecting to the cars service.

`oc exec "$(oc get pod -l app=travels -n user25-prod-travel-agency -o jsonpath={.items..metadata.name})" -c istio-proxy -n user25-prod-travel-agency -- openssl s_client -showcerts -connect $(oc -n user25-prod-travel-agency get svc cars -o jsonpath={.spec.clusterIP}):8000`

### Step 4 - Clean-up

Clean up the `PeerAuthentication` and re-run the above command to verify the mTLS configuration has been reinstated.

`oc delete peerauthentication cars-mtls-disable -n user25-prod-travel-agency`

# Part II: Setting Up Corporate CA for Service Mesh

## Task 4 (Optional): Use a custom Certificate Authority (CA)

This section is optional and you can also use for your own understanding of the procedure to provide custom CA certs.

### Step 1 - Create a custom CA and Intermediate CA Cert

Firstly you will create a custom CA and then with it an Intermediate CA (normally these will be provided).

- You must be in the directory of ***lab-5*** and provide to the following script the full path for this directory in your local system:
    
    `./create-corporate-intermediate-certs.sh /<YOUR-PATH>/lab-5`
    
- Follow the above scripts output and in the *verification* parts verify that:
    - a) The 2 `openssl.cnf` files have been created correctly with **`dir`** variable the path you provided:
    - b) `4a-index.txt` has content
        
        `V	330421081250Z		1000	unknown	/C=GB/ST=England/O=Travel Agency Ltd/OU=Intermmediate Certificate Authority/CN=www.travelagency.com/emailAddress=ca@www.travelagency.com`
        
    - c) *Verify the intermediate certificate* has
        - Issuer the custom *Certificate Authority*:
            
            `Issuer: C = GB, ST = England, L = London, O = Travel Agency Ltd, OU = Certificate Authority, CN = www.travelagency.com, emailAddress = ca@www.travelagency.com`
            
        - Subject is the *Intermediate Certificate Authority*:
            
            `Subject: C = GB, ST = England, O = Travel Agency Ltd, OU = Intermediate Certificate Authority, CN = www.travelagency.com, emailAddress = ca@www.travelagency.com`
            

### Step 2 - Verify the current controlplane and dataplane used certificates

The current *controlplane* and *dataplane* certificates are issued by default `Issuer: O = cluster.local`. Verify this with the following commands:

- Check the issuer of the default service mesh certificate (it should be something like `Issuer: O = cluster.local`)
    
    `./login-as.sh emma
    oc get -o yaml secret istio-ca-secret -n user25-prod-istio-system | grep ca-cert | awk '{print $2}' | base64 -d | openssl x509 -noout -text`
    
- Check the certificates used in the communication with `istiod`, the Issuer is again `Issuer=O = cluster.local`
    
    `oc exec "$(oc get pod -l app=istio-ingressgateway -n user25-prod-istio-system -o jsonpath={.items..metadata.name} | awk '{print $1}')" -c istio-proxy -n user25-prod-istio-system -- openssl s_client -showcerts -connect $(oc get svc istiod-user25-production -o jsonpath={.spec.clusterIP}):15012`
    
- Checking the certificates used in the POD to POD communications, the Issuer is again `Issuer=O = cluster.local`
    
    `oc exec "$(oc get pod -l app=travels -n user25-prod-travel-agency -o jsonpath={.items..metadata.name})" -c istio-proxy -n user25-prod-travel-agency -- openssl s_client -showcerts -connect $(oc -n user25-prod-travel-agency get svc cars -o jsonpath={.spec.clusterIP}):8000`
    

### Step 3: Modify the `production` SMCP tenant to use the corporate based CA, Intermediate CA and chain certificates

The [documentation](https://docs.openshift.com/container-platform/4.12/service_mesh/v2x/ossm-security.html#ossm-cert-manage-add-cert-key_ossm-security) instructs to create a secret named `cacert` that includes the input files `ca-cert.pem`, `ca-key.pem`, `root-cert.pem` and `cert-chain.pem`. Based on the files you generated the equivalent are:

- `intermediate.cert.pem` (`ca-cert.pem`). the certificate for the intermediate ca
- `intermediate.key.pem` (`ca-key.pem`). the key for the intermediate ca certificate
- `ca.cert.pem` (`root-cert.pem`): the root ca certificate
- `ca-chain.cert.pem` (`cert-chain.pem`)the certificate chain that includes both certificates
    
    `oc create secret generic cacerts -n user25-prod-istio-system \
    --from-file=ca-cert.pem=certs-resources/intermediate/certs/intermediate.cert.pem \
    --from-file=ca-key.pem=certs-resources/intermediate/private/intermediate.key.pem \
    --from-file=root-cert.pem=certs-resources/certs/ca.cert.pem \
    --from-file=cert-chain.pem=certs-resources/intermediate/certs/ca-chain.cert.pem`
    
- Remove the `istio-system-ca` secret created as default CA
by OSSM as it interferes with the istiod correctly picking the
enterprise certificates from the newly created `cacerts` secret
    
    `oc get  secret istio-ca-secret -n user25-prod-istio-system -o yaml > istio-ca-secret-default.yaml
    oc delete secret istio-ca-secret -n user25-prod-istio-system`
    
- Logged in as `Mesh Operator` (credentials `emma`/`emma`) add to the `ServiceMeshControlPlane` resource `user25-production` under `security` section the `certificateAuthority` properties as shown in the following example (make sure the indentation is correct). Service Mesh reads the certificates and key from the
secret-mount files.
    
    `./login-as.sh emma
    oc edit smcp/user25-production -n user25-prod-istio-system
    
    # Add in production SMCP under the existing _security_ section the followng:
    
      security:
        certificateAuthority:
          type: Istiod
          istiod:
            type: PrivateKey
            privateKey:
              rootCADir: /etc/cacerts`
    
- Restart *controlplane* and *dataplane* resources to force new certificate utilization
    - After adding the corporate the certificates, the control plane istiod and gateway pods must be restarted so the changes go into effect. Use
    the following command to restart the pods(The Operator will
    automatically recreate the pods after they have been deleted):
        
        `oc -n user25-prod-istio-system delete pods -l "app in (istiod,istio-ingressgateway, istio-egressgateway,gto-user25-ingressgateway)"
        oc -n user25-prod-istio-system get pods`
        
    - Restart the *dataplane* pods to expedite the sidecar proxies picking up the secret changes.
        
        `oc -n user25-prod-travel-control delete pods --all
        oc -n user25-prod-travel-agency delete pods --all
        oc -n user25-prod-travel-portal delete pods --all`
        
- Finally, perform some verifications of the new certificates being in effect
    - Identify the `cacerts` Issuer (it should be `C = GB, ST = England, L = London, O = Travel Agency Ltd, OU = Certificate Authority, CN = www.travelagency.com, emailAddress = [ca@www.travelagency.com](mailto:ca@www.travelagency.com)`)
        
        `oc get -o yaml secret cacerts -n user25-prod-istio-system | grep ca-cert | awk '{print $2}' | base64 -d | openssl x509 -noout -text`
        
    - Verify the *dataplane* communications are secured with the new corporate certificates
        
        `./login-as.sh emma
        ./verify-dataplane-certs.sh user25
        
        ###########################
         CERTS CHECK ON DATAPLANE
        ###########################
        
        1. Wait 20 seconds for the mTLS policy to take effect before retrieving the certificate chain of cars POD. As the CA certificate used in this example is self-signed, the verify error:num=19:self signed certificate in certificate chain error returned by the openssl command is expected.
        ------------------------------------------------------
        Can't use SSL_get_servername
        depth=2 C = GB, ST = England, L = London, O = Travel Agency Ltd, OU = Certificate Authority, CN = www.travelagency.com, emailAddress = ca@www.travelagency.com
        verify error:num=19:self signed certificate in certificate chain
        verify return:1
        depth=2 C = GB, ST = England, L = London, O = Travel Agency Ltd, OU = Certificate Authority, CN = www.travelagency.com, emailAddress = ca@www.travelagency.com
        verify return:1
        depth=1 C = GB, ST = England, O = Travel Agency Ltd, OU = Intermediate Certificate Authority, CN = www.travelagency.com, emailAddress = ca@www.travelagency.com
        verify return:1
        depth=0
        verify return:1
        DONE
        
        2. Parse the certificates on the certificate chain.
        ------------------------------------------------------
        
        3. Verify the root certificate used in the POD handshake is the same as the one specified by the OSSM administrator:
        ------------------------------------------------------
        Files /tmp/root-cert.crt.txt and /tmp/pod-root-cert.crt.txt are identical
        
        4. Verify the Intermediate CA certificate used in the POD handshake is the same as the one specified by the OSSM administrator:
        ------------------------------------------------------
        Files /tmp/ca-cert.crt.txt and /tmp/pod-cert-chain-ca.crt.txt are identical
        
        5. Verify the certificate chain from the root certificate to the workload certificate:
        ------------------------------------------------------
        ./proxy-cert-1.pem: OK`
        
    - Verify the *controlplane* communications are secured with the new corporate certificates
        
        `./verify-controlplane-certs.sh user25
        
        ###########################
        CERTS CHECK ON CONTROLPLANE
        ###########################
        
        1. Get the ceritificates used between istio-ingressgateway and istiod
        Can't use SSL_get_servername
        depth=2 C = GB, ST = England, L = London, O = Travel Agency Ltd, OU = Certificate Authority, CN = www.travelagency.com, emailAddress = ca@www.travelagency.com
        verify error:num=19:self signed certificate in certificate chain
        verify return:1
        depth=2 C = GB, ST = England, L = London, O = Travel Agency Ltd, OU = Certificate Authority, CN = www.travelagency.com, emailAddress = ca@www.travelagency.com
        verify return:1
        depth=1 C = GB, ST = England, O = Travel Agency Ltd, OU = Intermediate Certificate Authority, CN = www.travelagency.com, emailAddress = ca@www.travelagency.com
        verify return:1
        depth=0
        verify return:1
        DONE
        
        2. Verify the root certificate used in the istiod handshake is the same as the one specified by the OSSM administrator:
        ------------------------------------------------------
        Files /tmp/root-cert.crt.txt and /tmp/pod-root-cp-cert.crt.txt are identical
        
        4. Verify the Intermediate CA certificate used in the istiod handshake is the same as the one specified by the OSSM administrator:
        ------------------------------------------------------
        Files /tmp/ca-cert.crt.txt and /tmp/pod-cert-cp-chain-ca.crt.txt are identical
        
        5. Verify the certificate chain from the root certificate to the workload certificate:
        ------------------------------------------------------
        ./proxy-cp-cert-1.pem: OK`
        
        We have created an ebook resource which walks through the Day 2 Operations. You can download the [Getting started with Red Hat OpenShift Service Mesh](https://www.redhat.com/en/resources/getting-started-with-openshift-service-mesh-ebook)
        
        ## Day 2 Operations
        
        You will find in the associated github repository [ossm-heading-to-production-and-day-2](https://github.com/redhat-developer-demos/ossm-heading-to-production-and-day-2) instructions are the Day 2 Operations:
        
        - [Troubleshooting the Mesh](https://github.com/redhat-developer-demos/ossm-heading-to-production-and-day-2/tree/main/scenario-7-mesh-troubleshooting)
        - [Tuning the Mesh](https://github.com/redhat-developer-demos/ossm-heading-to-production-and-day-2/tree/main/scenario-8-mesh-tuning)
        - [Upgrade Mesh to a New Version](https://github.com/redhat-developer-demos/ossm-heading-to-production-and-day-2/tree/main/scenario-9-mesh-upgrade)