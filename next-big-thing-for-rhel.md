# What’s the next big thing for RHEL ?

Presenters: Chris Wells / Gunnar Helekson

Where Are We Going to Take It in the Future? What Is the Vision?

Mission Statement:

RHEL is the source for safe and reliable Linux innovation that makes your workloads successful. It can run everywhere - on-premises, virtual, in the cloud, and serves the Red Hat portfolio and the Red Hat ecosystem.

---

## Where We’ve Been

RHEL has more than 10,000 projects and is the foundation of the original hybrid cloud application platform. It supports everything as a service, modern apps, containers and microservices, and ISVs. The value of a Red Hat Enterprise Linux subscription includes support, security updates, and a partners ecosystem.

How It Started:

UNIX —> RHEL

## How It’s Going :

The majority of new workloads are using RHEL. It supports all cloud providers and partners, and the latest versions are RHEL 9.2 and 8.8. The goal is to make Linux simpler and more efficient, and to more easily manage production workloads. Broader support is provided for innovative new hardware, and resilient containers with automated podman health checks and improved event logging make it safer to deploy containers at scale.

Key Points for Those Who Want to Run Both Oracle Announcement:

Red Hat and Oracle will officially support joint customers in Oracle Cloud.

---

## RHEL Tomorrow

Library of trusted content, image builder, and host management tools (to find if they need a patch, do it from the website), system roles, and the next step is a disconnected environment.

Traditionally, RHEL is on a DVD. Now, it will create an image and deploy it.

 System roles protect us from major releases and updates. Management for all (insights) makes it easier to manage systems anywhere, simplifies end-to-end system management, and is delivered as a service - no infrastructure overhead. Interactive guidance is provided to lower the learning curve and is included with a subscription. Convert2RHEL is the way to migrate from CentOS to RHEL, and it now works on Azure/AWS/GCP so you don't have to destroy your RHEL7. It will be available in Q3.

---

## Where We’re Going

Step 1: Management for All

By using [cloud.redhat.com](http://cloud.redhat.com) as a single point for all your RHEL needs, it makes management really easy

Step 2: Moving Management from Run-Time to Build-Time

Most systems are run-time and they want to migrate that to build time. A management tool for image builds is being developed. RHEL will now come with an AI in image builder. For example, it will suggest other packages that are often installed with Nginx.

Step 3: Optimizing for Different Infrastructure

The hardware world is very diverse and a lot of architecture are popping up. One kernel cannot do everything. Specific kernels are being developed for 64-bit Arm, for example. There will be variations of RHEL (14 to 17 versions at any time), but it can be managed.

Step 4: Expanding Your Secure Supply Chain

There are currently 13,000 packages, but only 200 in the OS. Customers ask for new packages, so they need to keep adding them. The reason they don't do it today is that they have no tool. Now, they have the tool. They are aiming for a universe of software for apps, for infra, running and optimized on any infrastructure.

### How this could be useful to us :

We don’t use RHEL right now other than for our jumpoints, but that can be where we have something to gain. We could use the [cloud.redhat.com](http://cloud.redhat.com) interface to build an image that has everything we need for all jumpoints.