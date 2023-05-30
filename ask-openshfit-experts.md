# Ask the experts Redhat Openshift

Q&A type format. 

My questions :

Question: Can you give more information about how to use Hypershift with ACM in order to do ZTP ? Is it available in a disconnected environement

Answer: ZTP will go through metalkube to create a bootable ISO and install your nodes automatically from ACM. You can then use Hypershift to have your masters populate siteconfigs and create clusters on the fly with openshift virtualization.

What’s new in 4.13

crun, rhel 9, k8s 1.26

layered rpm’s

fips compliance in pipelines

namespace right sizing in ACM

Question : Wil OVS stop being supported in ARO as OVN is being more prominent ? 

Answer : OVN since 4.11 is the defacto choice and there should not impact cluster when you have to move to OVN. It will be non disruptive. 

---

Question : Can you explain more about right sizing. Can you force the customers to reduce the size of their pods.

Answer : Q1 next year. It uses prometheus to check for the best sizing in the namespace. 

---

Question : Why does ODF support only 1 tier of storage? Will muti-tier be supported

Answer : We don’t have the specific, IBM and redhat have merged together to bring more features. NVMe 

---

Question : Is there any manner to monitor openshift since in 4.10 the console was removed from grafana.

Answer : Prometheus is still there and you can Bring your own grafana. 

---

Question : Is there a way to know what Openshift is doing during an upgrade? I am running things like oc get nodes oc get co oc get clusterversion etcetc but am I doing it right?

Answer : There is a redhat insight project that is in developpement with IBM that will match all the telemetry and give you a note based on the chance of success for an upgrade. And they are looking into 

---

Question : Are you doing load testing on OVN in regards to ACL’S? we have a lot of ACL’s we had to rebuild ovn southbound 

Answer : 4.11 / 4.12 / 4.13 is where the biggest upgrades in terms of performance have been from OVN. 

---

Question : API management getting more integrated with a kubernetes clusters. 

Answer: there was a project recently to bring together projet quadrant taking the control plane to 3scale and bring it to envoy proxy.For mesh/ingress egress policys the next big thing is probably there. 

Istio team is taking the quadrant project and building on top of that. Ingress / east-west and api mangement policies. There is also skupper to link applications accross clusters 

---

Question : Can you use Citrix Netscaler

Answer : Yes there is an operator you can use to link OCP with citrix Netscaler. 

---

Question : Can you talk more about Sandbox containers 

answer: Peer pods to call an APi to manipulate where you want to run things like Kata containers. You can do nested virtualization

---

Question: