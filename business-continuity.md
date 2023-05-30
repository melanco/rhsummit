# achieving business continuity

This was a lab created by two gentlemen about disaster recovery using ACM/OCP/ODF and a new operator called ODR (Openshift disaster recovery)

The setup had a hub cluster with ACM on it to manage two other clusters, one cluster was called primary, the other secondary. 

The whole process is about 5 steps :

1- Deploy with ACM a new application on the primary cluster (in my case it was a store with a frontend, a database inside the cluster and a service/route to access it)

2 - Make sure I can use the DRPolicy resources in data storage on ACM cluster and make sure both clusters were linked in there via submariner. Create a data policy which sets the time at which is 

3- Make the priamry application failover to the secondary cluster  

4- Bring back the application on the primary cluster using ACM 

5- Create your own application and try it out

What I did is I created