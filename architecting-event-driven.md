# Architecting event-driven applications with OCP, CockroachDB and Kafka

What is EDA?

Event - Action driven or occurance recognized by software. 

Event-Driven Architecture (EDA) - is a way of designing applications and services 

## OpenShift Logins

In the terminal at the right, you will execute many of the command-line
instructions required for this lab. However, you must first log-in to OpenShift
as the correct user.

1. Login to OpenShift on the command line

`oc login -u user13 -p hMYJxIR3WQQ0DzJt`Note
The super security-conscious will recognize that entering passwords on the
command line as part of command execution is a bad practice.

2. Switch to your project

`oc project user13-eda`
3. Login to the OpenShift web console
Console URL: [https://console-openshift-console.apps.cluster-zj2lj.zj2lj.sandbox644.opentlc.com](https://console-openshift-console.apps.cluster-zj2lj.zj2lj.sandbox644.opentlc.com/)
    ◦ Login:

`user13`
    ◦ Password:

`hMYJxIR3WQQ0DzJt`
4. Switch to the Developer perspective
    ◦ In the OpenShift console’s left-side menu, select the *Developer* perspective
from the drop-down menu if it is not already selected.
    ◦ Then, make sure to select the `user13-eda` from the drop-down menu labeled
*Project* at the top-left.

![https://bookbag-user13-zj2lj-bookbag.apps.cluster-zj2lj.zj2lj.sandbox644.opentlc.com/workshop/images/choose-developer-perspective.png](https://bookbag-user13-zj2lj-bookbag.apps.cluster-zj2lj.zj2lj.sandbox644.opentlc.com/workshop/images/choose-developer-perspective.png)

## Fruit Shop Application

1. Next, you will deploy a pre-built container image for the Node.js "Fruit"
application.
    ◦ Select *+Add* in the left-hand navigation and then click the *Container Images*
tile.
    ◦ Enter the following image name in the box that says *Image name from external
registry*.

`quay.io/openshiftdemos/cockroach-kafka-eda-fruit-node:latest`
    ◦ Accept all of the other defaults and click the blue *Create* button at the
bottom.
    ◦ You will return to the *Topology* view and see that the pod comes up.
Click the *View Logs* under the *Pods* section.
You will see an error message that the env variable *SERVICE_BINDING_ROOT* is missing.
This is to be expected because the Node.js application expects a Service Binding to extract
the connection details to the database. So let us add the CockroachDB database instance and
create a Service Binding to it.

## CockroachDB Cloud Hosted Database

1. Next, you will add a CockroachDB Cloud Hosted Database
    ◦ On the OpenShift console’s left-side menu, select *+Add* and then click the *Cloud-Hosted
Database* tile.
    ◦ Select *CockroachDB Cloud* tile and then click the blue *Add to Topology* button.
    ◦ In the next screen select *+ Create New Database Instance* on the right-hand side.Warning
Make sure that you choose *+ Create New Database Instance*, otherwise you may
end up using someone else’s database. This OpenShift cluster is configured  with
a single database provider account for the CockroachDB Cloud, which is why you
may see all of the other databases.

    ◦ In the box that says *Cluster name*, enter `user13-db`.
    ◦ Scroll to the bottom and click the blue *Create* button.
    ◦ Click the blue "View Database Instances" button.
    ◦ Click the radio button for your database, `user13-db`.Warning
Make sure that you select your own database, and not a different user’s database.

    ◦ Click the blue *Add to Topology* button.
    ◦ Click the blue *Continue* button.
2. Create a service binding between the Node.js Fruit Shop application and
the database instance.
    ◦ Hover your mouse over the center of the blue pod ring. You will notice that a
small arrow appears coming out from the ring after a moment.
    ◦ Click and drag the arrow to the dotted-line box around the database.
    ◦ Click the blue *Create* button to accept the default service binding name.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c0e9b1c-b04f-4fbc-92c5-895140da49ab/Untitled.png)

A service binding in OpenShift automatically causes a Kubernetes `Secret` to be
injected into the pod. In your case, it is injecting a secret with the details
of the database connection. If you want to see the low-level implementation of
this behavior:

- In the *Topology* view, click on the pod ring for the Node.js app
- In the right-hand tab, click on the pod, which will have a name like `cockroach-kafka-eda-fruit-node-664b79b5d6-hv8jx`
- In the pod details screen, click on the *YAML* tab
- Scroll down to the `volumeMounts` section, which will look something like:
    
    `volumeMounts:
      - name: cockroach-kafka-eda-fruit-node-d-user12-db-5a60b62d6c-dbsc
        mountPath: /bindings/cockroach-kafka-eda-fruit-node-d-user12-db-5a60b62d6c-dbsc
      - name: kube-api-access-p4pkx
        readOnly: true
        mountPath: /var/run/secrets/kubernetes.io/serviceaccount`
    

The Node.js application knows how to extract the relevant connection details from
this mounted secret.

After creating the service binding, the pod will restart and get to a *running* state
indicated by the blue ring. The pod log will be free of any Service Binding error.

## Testing

1. Test the Fruit Shop application
    ◦ Open Application UI by clicking the route opener
    ◦ Perform basic tasks like inserts/updates within the application UI. For
example, add 2 Strawberries.
2. Obtain credentials to connect to CockroachDB Serverless cluster from Service
binding
    ◦ On the Topology page click on the Service Binding arrow:
    ◦ In the new right-hand panel, click on the link for the Secret:
    ◦ From the Service Binding Secret page, click the *Reveal values* link
    ◦ Observe the values for various connection string parameters, but you will use
a little bash scriptlet to do the hard work for you.
    ◦ The `cockroach` CLI is embedded in the terminal. The following bash script will use `jq` to extract details from the secret, and then build the connection string:

`cat ~/assets/test-service-binding.sh`
    ◦ Execute the scriptlet using the following command, and note that it 
prints out the password you will need to type or copy/paste into the 
CLI:

`bash ~/assets/test-service-binding.sh`
    ◦ You will see output like the following:

`Use this password: AmR6x~3C(Cd/
#
# Welcome to the CockroachDB SQL shell.
# All statements must be terminated by a semicolon.
# To exit, type: \q.
#
Connecting to server "free-tier14.aws-us-east-1.cockroachlabs.cloud:26257" as user "user1_eda.user1_db_a8375c8343".
Enter password:`
    ◦ Validate the data was stored in the Database using basic SQL commands:

`select * from fruit limit 5;`

```yaml
[~] $ bash ~/assets/test-service-binding.sh
Use this password: (z})L08O#rv@
#
# Welcome to the CockroachDB SQL shell.
# All statements must be terminated by a semicolon.
# To exit, type: \q.
#
Connecting to server "free-tier14.aws-us-east-1.cockroachlabs.cloud:26257" as user "user13_eda.user13_db_2c340c2b86".
Enter password:
# Client version: CockroachDB CCL v22.2.7 (x86_64-pc-linux-gnu, built 2023/03/28 19:47:29, go1.19.6)
# Server version: CockroachDB CCL v22.2.9 (x86_64-pc-linux-gnu, built 2023/05/08 12:58:16, go1.19.6)
# Cluster ID: 2b8401dd-b4cc-4266-4f1a-6cc21febcee9
#
# Enter \? for a brief introduction.
#
user13_eda.user13_db_2c340c2b86@free-tier14.aws-us-east-1.cockroachlabs.cloud:26257/defaultdb>
user13_eda.user13_db_2c340c2b86@free-tier14.aws-us-east-1.cockroachlabs.cloud:26257/defaultdb>
user13_eda.user13_db_2c340c2b86@free-tier14.aws-us-east-1.cockroachlabs.cloud:26257/defaultdb> select * from fruit limit 5
;
                   id                  |   name    | quantity |      description
---------------------------------------+-----------+----------+------------------------
  51661376-0a07-449b-a3bd-9db79cd4ead4 | Apple     |        6 | Keeps the doctor away
  69f6cd81-59fc-493b-8ebf-1b9f150ecead | Blueberry |        8 | Antioxidant Superfood
  d37f4fae-b572-47b3-93e0-17daf798f9d5 | Banana    |        7 | Good for health
(3 rows)

Time: 14ms total (execution 1ms / network 13ms)

user13_eda.user13_db_2c340c2b86@free-tier14.aws-us-east-1.cockroachlabs.cloud:26257/defaultdb>
```

    ◦ Exit the CockroachDB CLI:

`quit`

# Kafka

## Create the Kafka Topic

Now that your basic application components are deployed, you will 
create a Kafka topic that will be used for streaming the events.

1. You will create your Kafka topic using the `oc` CLI. The following YAML has been staged for you:

`apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: userX-table-changes
  labels:
    strimzi.io/cluster: crdb-cluster
  namespace: userX-test
spec:
  config:
    retention.ms: 604800000
    segment.bytes: 1073741824
  partitions: 1
  replicas: 3
  topicName: userX-table-changes`
    ◦ Execute the following command which will substitute your correct username into
the YAML before passing it to `oc`:

`sed 's/userX/user13/' ~/assets/kafkatopic.yaml | oc create -f -`

## Test the Kafka Topic

There are two small Python applications that you will use to test the Kafka
topic.

1. Deploy the Producer application
    ◦ Create the Kafka Producer application by clicking *+ Add* in the left-hand
navigation
    ◦ Click the *Container images* tile
    ◦ Use the following image name in the *Image from external registry* box:

`quay.io/openshiftdemos/python-kafka-producer:latest`
    ◦ Scroll down to the text that says *Click on the names to access advanced
options* and click the word *Deployment*
    ◦ Add an environment variable called:

`TOPIC_NAME`
with the value:

`user13-table-changes`
    ◦ Click the blue *Create* button.
2. Initialize the Topic
    ◦ Click on the Producer application’s route, which will open a new browser tab.
You will see something like the following:

`{"kafka_produce":"posted messages"}`

Don’t close this window, but go back to the OpenShift web console.

1. Deploy the Consumer application
    ◦ Create the Kafka Consumer application by clicking *+ Add* in the left-hand
navigation
    ◦ Click the *Container images* tile
    ◦ Use the following image name in the *Image from external registry* box:

`quay.io/openshiftdemos/python-kafka-consumer:latest`
    ◦ Scroll down to the text that says *Click on the names to access advanced
options* and click the word *Deployment*
    ◦ Add an environment variable called:

`TOPIC_NAME`
with the value:

`user13-table-changes`
    ◦ Click the blue *Create* button.
2. Test the Producer application
    ◦ Click on the Consumer pod in the *Topology* view.
    ◦ In the right-hand navigation, click into the pod in the Pods list. It will
have a name like `python-kafka-consumer-696d6fbfbb-r9kk6`
    ◦ Click on the *Logs* tab in the pod’s details window
    ◦ Note that the Consumer’s log indicates that your earlier visit to the Producer
produced messages.
    ◦ You can switch between the Producer application’s tab, refreshing the browser,
and the Consumer pod’s logs to watch the messages.
3. Delete the Producer application
    ◦ The producer application is no longer needed, so you can delete its assets:

`oc delete all -l app=python-kafka-producer`

| Note | There is a feature in the OpenShift console when using the + Add workflow to
use application grouping which would have made deleting the components easier.
If you click the three dots next to one of the deployment names in the topology,
you will see an option called Edit application grouping. Feel free to try
moving the Kafka consumer to its own group. |
| --- | --- |

You will leave the Consumer application running as it will be used later for
debugging.

# Configuring the Changefeed

## CockroachDB and Kafka

CockroachDB is capable of acting as a Kafka producer that publishes messages to
Kafka topics in relation to database events. As this lab uses the CockroachDB
Cloud, you will need to configure the Kafka (running in the OpenShift cluster)
to allow external, authenticated communication.

## Configuring Kafka for External Communication

When the lab environment was set up, the Kafka cluster was exposed externally
via an OpenShift Route. The route was secured using TLS encryption and
communicates as HTTPS over port 443 (the default).

To get the route for the Kafka bootstrap cluster, you have two options.

### Option 1: OpenShift GUI

1. Click the *Project* dropdown at the top of the *Topology* area and choose the `crdb-kafka` project
2. Click *Project* in the left-hand navigation
3. Click *Routes* in the *Inventory* area
4. Find the *Location* for the route called `crdb-cluster-kafka-external-bootstrap`

### Option 2: OpenShift CLI

The following command will get the route information:

`oc get route crdb-cluster-kafka-external-bootstrap -n crdb-kafka`

Because the URL is so long, the line wrapping makes things look a little messy.
The important details are the following:

- the hostname, which ends in .com in our case
- the termination type, which is *passthrough*, meaning that the HAProxy load
balancer handling the external connections to OpenShift will pass the TLS
connection through to the endpoint (Kafka, in this case) without re-encrypting
or de-encrypting

## Obtaining the TLS certificate

CockroachDB, like most TLS-enabled applications, needs the certificate in order
to properly communicate.

While you could also find the certificate by digging in the OpenShift web
console, it’s fairly quick to do it via the CLI:

`oc describe kafka crdb-cluster -n crdb-kafka`

You will see something like:

`Name:         crdb-cluster
Namespace:    crdb-kafka
Labels:       <none>
Annotations:  <none>
API Version:  kafka.strimzi.io/v1beta2
...
...
    Certificates:
      -----BEGIN CERTIFICATE-----
MIIFLTCCAxWgAwIBAgIUcFEfRknbqi5qbQnelpfNjKEQV4cwDQYJKoZIhvcNAQEN
...`

## Set up the CockroachDB changefeed

Setting up the changefeed for your CockroachDB Cloud instance to talk to the
Kafka broker is as easy as an SQL command. Fortunately, we have provided a small
bash script that will save you the trouble of lots of copying and pasting:

`cat ~/assets/cockroach-changefeed.sh`

When you run the above scriptlet, the route and certificate details will be
fetched and then passed to the CockroachDB CLI. Just copy and paste the password
shown.

`bash ~/assets/cockroach-changefeed.sh`

If successful, you should see something like:

`Use this password: i33$ylgU7azN
Connecting to server "free-tier14.aws-us-east-1.cockroachlabs.cloud:26257" as user "user1_eda.user1_db_a8375c8343".
Enter password:
        job_id
----------------------
  860339889937547265
(1 row)

Time: 127ms

NOTICE: changefeed will emit to topic user1-table-changes`

## Verify the CockroachDB changefeed

Even though the above output is pretty clear about whether or not is successful,
you can additionally verify the changefeed with an SQL command. Reconnect to the
database using the service binding test script:

`bash ~/assets/test-service-binding.sh`

Once connected, the following SQL will tell you the status of any changefeed
configuration:

`show changefeed jobs;`

Somewhere buried in that output you should see:

`CREATE CHANGEFEED FOR TABLE`

Exit the CockroachDB client:

`quit`

## Test the CockroachDB changefeed with the Fruit app

Make sure you have the Python consumer application’s logs showing:

1. In the OpenShift web console, ensure you are using the *Developer* perspective
2. Make sure that you are in the *Topology* view
3. Select your `user13-eda` project from the dropdown at the top
4. Click the Python consumer application pod
5. Click the pod name in the right hand navigation, which looks like
`python-kafka-consumer-696d6fbfbb-cfxw8`
6. Click the *Logs* tab

You will want to open another browser window into the OpenShift console to find
the Fruit application’s route.

Alternatively, you could watch the Python consumer application’s logs via the
OpenShift command line:

`oc logs -f `oc get pod -l app=python-kafka-consumer -o jsonpath='{.items[0].metadata.name}'``

| Note | You could have simply done an oc get pod to determine the name of the
consumer’s pod, and then done oc logs -f <podname> - but the scriptlet is more
convenient! |
| --- | --- |

Return to the Fruit application by clicking its route button in the *Topology*
view of your project:

Manipulate some fruit, and then watch the Python consumer application. You
should see that changes are flowing through to it via Kafka! If you created
*Strawberries* with a quantity of *6*, you might see:

`Received message: {"after": {"description": null, "id": "6e154890-764e-449b-b6d0-f57216c908d9", "name": "Strawberries", "quantity": "6"}}`

Press `Ctrl-C` to stop watching the logs if you chose to do so in the terminal
window.

Exemple de log :

```yaml
Received message: Kafka Test Message 19
Received message: {"after": {"description": "Good for health", "id": "d37f4fae-b572-47b3-93e0-17daf798f9d5", "name": "Banana",
"quantity": "7"}}
Received message: {"after": {"description": null, "id": "37700b23-1d68-44c9-a686-70507a77ed93", "name": "Strawberry", "quantity
": "9"}}
Received message: {"after": {"description": "Keeps the doctor away", "id": "51661376-0a07-449b-a3bd-9db79cd4ead4", "name": "App
le", "quantity": "6"}}
Received message: {"after": {"description": "Antioxidant Superfood", "id": "69f6cd81-59fc-493b-8ebf-1b9f150ecead", "name": "Blu
eberry", "quantity": "8"}}
Received message: {"after": {"description": null, "id": "ca423afb-6e9d-4caa-97bf-7d90d8699737", "name": "Raspberry", "quantity"
: "15"}}
```

# EDA Consumer Microservice

## Accesing the sample application in Gitea

There is a Gitea application, which is an open-source Github-like solution,
running inside the cluster. You can access it at the following URL:

`https://gitea-gitea.apps.cluster-zj2lj.zj2lj.sandbox644.opentlc.com`

Your Gitea username is:

`user13`

And the password is:

`openshift`

Sign in to Gitea clicking the link at the upper-right.

When you sign in to Gitea, you will see that you already have a repository
staged for you, called `user13/cockroach-kafka-eda`. Click into it now.

In preparation to do some light coding and process some Kubernetes/OpenShift
manifests, it will be necessary to clone this repository:

`cd ~;\
git clone https://gitea-gitea.apps.cluster-zj2lj.zj2lj.sandbox644.opentlc.com/user13/cockroach-kafka-eda;\
cd ~/cockroach-kafka-eda
oc project user13-eda`

## OpenShift Pipelines and Pipeline resources

OpenShift includes a CI/CD solution, OpenShift Pipelines, that is based on the
upstream Tekton project. The repository you cloned includes some Pipeline
manifests for building your event-driven application source. Please create those
resources now:

`oc create -f pipelines/tasks.yaml -f pipelines/pipeline.yaml`

You will see output like the following:

`task.tekton.dev/apply-manifests created
task.tekton.dev/update-deployment created
task.tekton.dev/create-cm created
pipeline.tekton.dev/build-and-deploy-ms created`

| Note | If you get an "already exists" error, it might mean you just executed the create
command twice. No worries! |
| --- | --- |

1. Now, go back to the OpenShift web console, making sure you are using the
`user13-eda` project
2. In the left-hand navigation, choose the *Administrator* perspective at the
top.
3. In the left-hand navigation, Click *Pipelines*
4. Click *Tasks*

You should see the three tasks that you created:

| Note | You can also find the tasks from the developer perspective by choosing Search
and then choosing the Task resource type, but this way is a little easier. |
| --- | --- |

1. Return to the *Developer* perspective
2. Click *Pipelines*

You should see the pipeline you created:

Click on the pipeline, and you will see a graphical depiction of the various
tasks and how they are ordered and their dependencies:

## Run the Pipeline

Now you will run the pipeline you created. While you could start the pipeline
run by clicking *Actions* and then *Start*, the pipeline run requires a large
number of parameters to be set, and it’s easier to do this from the command
line.

Go ahead and start the pipeline run with the following command:

`tkn pipeline start build-and-deploy-ms \
-w name=shared-workspace,volumeClaimTemplateFile=pipelines/pipelinepvc.yaml \
-p deployment-name-p=eda-producer-ms-ep \
-p deployment-name-c=eda-consumer-ms-ep \
-p git-url=https://gitea-gitea.apps.cluster-zj2lj.zj2lj.sandbox644.opentlc.com/user13/cockroach-kafka-eda.git \
-p IMAGE-P=image-registry.openshift-image-registry.svc:5000/user13-eda/eda-ms-producer \
-p IMAGE-C=image-registry.openshift-image-registry.svc:5000/user13-eda/eda-ms-consumer \
-p KAFKA_BROKER='crdb-cluster-kafka-bootstrap.crdb-kafka.svc.cluster.local:9092' \
-p KAFKA_GROUP_ID=user13-groupid \
-p KAFKA_TOPIC=user13-topic \
-p KAFKA_CLIENT_ID=user13-clientid \
--use-param-defaults`

1. In the OpenShift web console, return to the *Pipelines* menu
2. Click the `build-and-deploy-ms` pipeline
3. Click the *PipelineRuns* tab
4. Click on the pipeline run

You can now watch the pipeline do its thing. As each task completes, it will get
a green checkmark. If you click the *Logs* tab, you can observe the log output
of each of the tasks.

While you’re waiting for the pipeline run to finish, here’s an explanation of
the `tkn` command above:

- Start the `build-and-deploy-ms` pipeline
- Use some shared storage, name it `shared-workspace`, and use a Kuberentes
volume claim based on the YAML to determine the size and other characteristics
of the storage
- The git URL for the source code
- The names of the producer and consumer resources
- The output destination for the built images in the OpenShift container image
registry
- Some parameters for the apps to talk to Kafka

The entire pipeline run may take six or seven minutes, so, if you’re a fast
reader, now would be a great time to relax your eyes, stand up and stretch, or
play with your phone. We’ll wait.

## Expose the deployed applications

Routes (and Kubernetes Ingresses) make applications available outside the
cluster. Go ahead and execute the following commands to expose the applications
you built and deployed with the pipeline:

`oc create route edge --service=eda-consumer-ms-ep-svc -n user13-eda
oc create route edge --service=eda-producer-ms-ep-svc -n user13-eda`

You will see output like:

`route.route.openshift.io/eda-consumer-ms-ep-svc created
route.route.openshift.io/eda-producer-ms-ep-svc created`

The `--edge` flag tells the command line tool to create a route that uses TLS
edge termination. This means that TLS encryption stops at the OpenShift router
(HAProxy) and is then un-encrypted between the router and the pod(s).

The following command will give you more details about one of the routes you
just created:

`oc describe route eda-consumer-ms-ep-svc`

You will see output like:

`Name:                   eda-consumer-ms-ep-svc
Namespace:              user1-eda
Created:                2 minutes ago
Labels:                 app=eda-consumer-ms-ep
Annotations:            openshift.io/host.generated=true
Requested Host:         eda-consumer-ms-ep-svc-user1-eda.apps.cluster-62k9w.62k9w.sandbox2634.opentlc.com
                          exposed on router default (host router-default.apps.cluster-62k9w.62k9w.sandbox2634.opentlc.com) 2 minutes ago
Path:                   <none>
TLS Termination:        edge
Insecure Policy:        <none>
Endpoint Port:          <all endpoint ports>

Service:        eda-consumer-ms-ep-svc
Weight:         100 (100%)
Endpoints:      10.129.2.46:3000`

## Test the deployed producer

To test the producer, you will want to look at the logs. You can either do this
by visiting the OpenShift web console, or using the CLI. The lab guide will show
you the CLI version.

In the web console’s *Topology* view, the producer application is called
`eda-producer-ms-ep`. Do you remember how to find the logs for its pod? If not,
look at the previous lab exercises.

Execute the following curl to hit the producer application:

`curl https://$(oc get route eda-producer-ms-ep-svc -o jsonpath='{.spec.host}')/produce`

You will see output like:

`{"message": "Requested to produce sample messages on user1-topic topic" }`

Now, check the producer logs:

`oc logs $(oc get pod -l app=eda-producer-ms-ep -o name)`

You should see a bunch of references to the following:

`produced to the topic user1-topic`

## View the Consumer application

You can get to the routes for the various application components from the
*Topology* view, or you can use the following command to get the URL:

`oc get route eda-consumer-ms-ep-svc`

You can copy/paste the hostname, and don’t forget the HTTPS! Or you can use the following bash-fu:

`echo "https://$(oc get route eda-consumer-ms-ep-svc -o jsonpath='{.spec.host}')"`

That’s a lot of pears.

## Bonus Round! It’s actually broken!

If you open the browser console, you’ll see that the websocket connection that
the consumer application is making is failing. That’s because the source code
has a hard-coded URL and doesn’t use any kind of environment variable or other
parameter.

Take a look at the source code for the consumer webpage:

`cd ~/cockroach-kafka-eda
cat consumer/test.html | grep replaceme`

It should be obvious what’s wrong:

    `webSocket = new WebSocket("wss://replacemewithconsumerurl/foo");
      fetch('https://replacemewithproducerurl/produce')`

Now, ideally you would be using a dynamic application where the server would
interpret its environment variables and would determine the endpoint for the
websocket connection in real time before serving the page to the client.
However, this is a simple single-page HTML "application" so you’ll have to
hard-code the websocket endpoint URL in the HTML file.

First, fix the consumer URL:

`export CONSUMER_HOST=$(oc get route eda-consumer-ms-ep-svc -o jsonpath='{.spec.host}')
sed -i "s/replacemewithconsumerurl/$CONSUMER_HOST/" ~/cockroach-kafka-eda/consumer/test.html`

Then, fix the producer URL:

`export PRODUCER_HOST=$(oc get route eda-producer-ms-ep-svc -o jsonpath='{.spec.host}')
sed -i "s/replacemewithproducerurl/$PRODUCER_HOST/" ~/cockroach-kafka-eda/consumer/test.html`

Check your work:

`grep 'WebSocket|fetch' ~/cockroach-kafka-eda/consumer/test.html`

You should see something like:

    `//webSocket = new WebSocket("ws://localhost:3000/foo");
    webSocket = new WebSocket("wss://eda-consumer-ms-ep-svc-user1-eda.apps.cluster-rr4l2.rr4l2.sandbox1899.opentlc.com/foo");
      fetch('https://eda-producer-ms-ep-svc-user1-eda.apps.cluster-rr4l2.rr4l2.sandbox1899.opentlc.com/produce')`

If you don’t, feel free to use `vi` or `nano` to edit the `test.html` file
directly to fix things. Don’t forget to save.

In order to commit your changes back to the git repo, you’ll need to configure
the terminal’s `git` client. The following can be used:

`git config --global user.email "user13@example.com"
git config --global user.name "user13"`

Then, commit your changes:

`git commit -am "fixing websocket URLs"`

Finally, push your code:

`git push`

Your Gitea username is:

`user13`

And the password is:

`openshift`

Feel free to visit Gitea (using the URL at the beginning of the lab) to see
your changes.

Now, you can trigger another pipeline run:

`cd ~/cockroach-kafka-eda
tkn pipeline start build-and-deploy-ms \
-w name=shared-workspace,volumeClaimTemplateFile=pipelines/pipelinepvc.yaml \
-p deployment-name-p=eda-producer-ms-ep \
-p deployment-name-c=eda-consumer-ms-ep \
-p git-url=https://gitea-gitea.apps.cluster-zj2lj.zj2lj.sandbox644.opentlc.com/user13/cockroach-kafka-eda.git \
-p IMAGE-P=image-registry.openshift-image-registry.svc:5000/user13-eda/eda-ms-producer \
-p IMAGE-C=image-registry.openshift-image-registry.svc:5000/user13-eda/eda-ms-consumer \
-p KAFKA_BROKER='crdb-cluster-kafka-bootstrap.crdb-kafka.svc.cluster.local:9092' \
-p KAFKA_GROUP_ID=user13-groupid \
-p KAFKA_TOPIC=user13-topic \
-p KAFKA_CLIENT_ID=user13-clientid \
--use-param-defaults`

Return to the OpenShift web console to view the pipeline run’s status. Note that
OpenShift will keep track of previous pipeline runs. This can be a very valuable
tool for debugging and/or understanding the quality, fragility, and overall
success of your CI/CD pipelines.

Revisit the consumer application:

`echo "https://$(oc get route eda-consumer-ms-ep-svc -o jsonpath='{.spec.host}')"`

You should see the bars change after a few moments, and see no errors in the
browser console. Hit the "Produce Events" button, and you should see things
change in the bars.