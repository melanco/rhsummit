# Compliance as code

Secure multicloud environements with ACM, ACS argocd and Kyverno

Speaker :

Tommer Amber, principle consultant

---

Agenda

problem

define the soluton

acm & openshift-gtiops

acm & kyverno

acm & acs

Main point

implement sucurity compliance on many clusters.

The problem

Not so long ago 

Security used to be simple, 

Moving to openshift

zero trust 

moving to multiple clusters on multiple vendors

Common use-cases

for every env

other scenarios :

multisite/dr/edge clusters

our solution

requirement list

- centrail management tool
- policies - easy to create and store in a single point
- Scalable (enforce on new clusters at ease)
- Flexible & controllable
- observability - compliance status & drifts

One diagram picture

Centrail management tool

ACM is the tool 

ACM is very good for security integrated. 

ACM gouvernance policiy engine 

Security policies are in yaml and acm will enforce them on all the clusters.

acm policy object (state of the object to enforce in clusters)

placement rule (where)

policy types

Example picture kubeadmin

inform or enforce

placement rule picture

cluster slector key/value

the added value

all policies are yaml based

pôlicy can be adjusted to enforce or inform

policiy will only affect the cluster with required labels

we can group multiple oplicies under the same label to create a new benchmark

acm reports in ACM UI for non compliant polciies

How should you mange polciies

with openshift git ops

write commit to git repo (for policies only)

gitops is installed on the same cluster as ACM will fetch polcies and ACM will deploy the polciies

acm can install tools for us

Acm can install kyverno and deploy kyverno policies on the remote cluster.

Bank of pre-configured polciies on open-mamangement (open source acm)

still need :

better flexibility

application security

ACM only solves infra security and this is where ACS will come in handy.

Kybverno for applcaition and 

if a developpeur tried to create a service the admission controller will allow 

swervice clusterIP is ok but nodeport no.

Kyverno quick overview

100% cloud native (standard yaml)

Kyverno policies can be encapsulated inside acm policies

quick and adjustable with helm

does all the k8s heavy lifting for us 

auto-create vialation reports 

multi polciies types

validation - yes/no

mutation - proactively esit object if not match condition

creation. - add an object automatically once a condition is met

can enforce polciies on existe project & newly create projects.

bank of pre-configured polciies (kyverno bank of polciies)

If I have acm polciies why do I need kyverno

Maximum flexibility

exemple :

disallow-latest-tag example

we are practically scripting polciies which is more flexible tahnacm polciies

once an object is tried to be created, kyverno will audit it but it can enforce it and if it does the message will be sent to the developper. 

ACM & kyverno how it works

acm will deploy the kyverno 

example

gt the youtube video and the github

label in ACM the label for it to manage install-kyverno

policyreport (oc get policiyreport)

to get the status of policies violation in auditing.

finalpiece

Secure application with ACS

picture

acm & acs

demo 

violation is the more powerful page of the product

CVE’s high // medium

Compliance can show how the cluster 

Network graph can show every pod communication

Simulare network policy and place it in cluster.

you can simulate the effect.