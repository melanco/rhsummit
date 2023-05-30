# Kubernetes operational excellence using gitOps

Openshift, ansible and ACM at morgan stanley.

Speakers :

Arvin Amirian 

Alishia Gryniewicz

---

Their environnement :

on prem / public cloud azure and aws

morgan stanley background is a bank. They are global leaders in finanacial. 

53.6 billion in revenue, 82000 employees, 20000 are IT. 

5.5 trillion worth of assets at any given time.

Industry challenge

Do more with less (budget)

Talent acquisition and retention

All firms competing for the same domain sme’s

Migration from legacy to modern

Secure and complant container Orchestration at scale.

Legacy environnement

Windows, linux, solaris

mainframe, physical, virtual

Proprietary, software distribution, container orchestration batch and scheduling

Target :

cloud native

lower total cost and maintenance

scalability&stability

desire to leverage existing DC’s and build new

Enterprise grade private cloud in 10 countries.

Flexibility protability 

security and compliance

highly regulated industriy across multiple contries.

---

Architecture 

infra clusters baremetal 3 masters/3 computes each in diffrent racks two nics to seperate TOR switches

Openshift vitualization used to host vms

no application workloads

infrastructure pod

related set of infra 

tenant clusters

3 virtualized master nodes

baremetal compute nodes

app teams procure right sized baremetal

high resiliency

smaller tenant clusters

full recovery from an infra cluster failure within 2 hours *excerise regularly)

datacenter density maximize compute capacity

controlled via single pane of glass

lower cost of operation

---

gitOps everywhere

Ansible/ RHACM deployment

All cluster configurations including external dependencies

Deployments managed by ansible config

Guaranteed consistency of config in clusters

allows for applications to be the same everywhere

scaling quickly was the requirement. 

Reduction of SRE’s toil

Scaling exponentially without increasing operational resources

Reducing SRE’s allows them to spend time on tasks that provide ROI’s.

Automation of adjacent infra

Common experience for applciation deplyment on kuibernetes 

technical challenges

---

Upgrades

SRE toil coordinate clusters owners for SRE to kick off and monitor jobs

rapid developement on kubernetes space

Unified control mechanism for cluster configs

stateful workloads

More detail :

upgrades 

high automation complexity of upgrading a matrix of products

openshift/calico/trident/rrhacm/openshift virt

change of config deviation

switch from upgrades to rebuild (nuke and pave)

application workload automatically deployed on cluster after cluster rebuild. 

flux on every cluster in prod and git. 

new openshift version every 4 months

it’s a challenge to automate and test every Y release

Inevability there are version compatibility issues integrating OCP/calico /trident.

Use EUS releases

Allows for 2 release hops of Openshift

Support n2 release at any given time.

SRE toil

Manual health checs of cluster post deploymenty

manually execution of deployment automation

must coordinate with cluster onwer when rebuilding for updates

Cluster health check automation via ansible

Self-service model for cluster rebuilds for cluster owners

AAP executes taks triggered bu service now ticket.

stateful workloads

Legacy workloads migration to lubernetes have persisten sortage reuqietsment tjat cournter k8s design

PB data should be automatically backedup

PV data should be automatically retored on cluster rebuild

Provides steppingstone to help existing application modernize

Summary ;

architecture and processes that meets the requirements of the stakeholders. 

gitops deployment policies to ensure engineering creates reeatable automation

leverage best of breed tooling for each component architecture from automation to runtime.

priority classes for storage. 

how long did the journey take 

about 3 years

size of team

about 20 people

networking team validates the calico

storage team wrote the trident restoration piece. 

ODF

they are working on it as they speak. 

---

Unique architecture