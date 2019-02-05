RHV 4.2 Disaster Recovery - LAB GUIDE
==================================

Deployment
----------

* Click on the GUID Generator Link to generate your GUID for the lab: 

[GIUD Generator](https://www.opentlc.com/gg/gg.cgi?profile=generic_emea_jskorzyn)

* Choose lab from the list: **RHV 4.2 Disaster Recovery LAB**

* Enter LAB Activation Key

Wiil be provided by instructor

[![Lab List](https://github.com/jskorzyn/RHV-4.2-LABs/blob/master/files/lab02.png)](https://www.opentlc.com/gg/gg.cgi?profile=generic_emea_jskorzyn)


Preparation
-----------

1.  Create bookmarks in your browser for:

| Name                                  | URL                                   |
|--------------------------------------:|--------------------------------------:|
| RHV PROD Administration Portal        | [https://rhvm-$GUID.rhpds.opentlc.com](https://rhvm-$GUID.rhpds.opentlc.com) |
| RHV DR Administration Portal          | [https://dr-rhvm-$GUID.rhpds.opentlc.com](https://rhvm-$GUID.rhpds.opentlc.com) |
| Ansible Tower                         | [https://tower-$GUID.rhpds.opentlc.com](https://tower-$GUID.rhpds.opentlc.com) |
| Survey direct link (TBD)              | [http://survey-$GUID.rhpds.opentlc.com/survey/<TBD>](https://survey-$GUID.rhpds.opentlc.com%2Fsurvey%2F%3CTBD%3E) |
| Survey settings page (TBD)            | [https://survey-$GUID.rhpds.opentlc.com/survey/<TBD>](https://survey-$GUID.rhpds.opentlc.com%2Fsurvey%2F%3CTBD%3E) |
| Survey statistics page (TBD)          | [https://survey-$GUID.rhpds.opentlc.com/survey/<TBD>](https://survey-$GUID.rhpds.opentlc.com%2Fsurvey%2F%3CTBD%3E) |

2.  Access RHVM and start the following VM:
    1.  Survey

Pre-flight Checks
-----------------

Some verifications should be done before starting the demonstration, to ensure everything works as expected.

1.  Access RHV Manager on PROD to accept TLS certificates
2.  Access RHV Manager on DR to accept TLS certificates
3.  Access Tower interface to accept TLS certificates
4.  Access survey admin application  to accept TLS certificates ([https://fqdn/survey/admin](https://mojo.redhat.com/fqdn/survey/admin))
5.  Ensure published survey is available
6.  Ensure “infra” storage domain is the Master SD in RHV PROD. If not, make it be taking “GlusterSD1” into maintenance and then activating it again.
7.  Ensure the Data Centers in both “sites” are up.
8.  Ensure geo-replication is active from primary site to secondary (gluster01.example.com → dr-gluster01.example.com) - use command line

Credentials
-----------

| Name              | User Name     | Password                    |
|------------------:|:-------------:|:----------------------------|
| RHV Admin Portal  | admin         | r3dh4t1!                    |
| Ansible Tower     | dr-admin      | r3dh4t1!                    |
| RHV hosts         | root          | r3dh4t1!                    |
| All VMs           | admin         | r3dh4t1! (with sudo access) |
| LimeSurvey app    | admin         | r3dh4t!                     |


--------------------

Lab Instructions
--------------------
### A Bit Of Theory

Red Hat Virtualization (RHV) supports two types of disaster recovery solutions to ensure that environments can recover when a site outage occurs. Both solutions support two sites, and both require replicated storage. 

#### Active-Active Disaster Recovery Scenario

Red Hat Virtualization supports an active-active disaster recovery failover configuration that can span two sites. Both sites are active, and if the primary site becomes unavailable, the Red Hat Virtualization environment will continue to operate in the secondary site to ensure business continuity.

The active-active failover is achieved by configuring a stretch cluster where hosts capable of running the virtual machines are located in the primary and secondary site. All the hosts belong to the same Red Hat Virtualization cluster. 

##### Stretch Cluster Configuration

![](https://github.com/jskorzyn/RHV-4.2-LABs/blob/master/files/RHV_Disaster-Recovery-Active_01.png)


Virtual machines will migrate to the secondary site if the primary site becomes unavailable. The virtual machines will automatically failback to the primary site when the site becomes available and the storage is replicated in both sites. 

##### Failed Over Stretch Cluster

![](https://github.com/jskorzyn/RHV-4.2-LABs/blob/master/files/RHV_Disaster-Recovery-Active_02.png)

#### Active-Passive Disaster Recovery Scenario

Red Hat Virtualization supports an active-passive disaster recovery solution that can span two sites. If the primary site becomes unavailable, the Red Hat Virtualization environment can be forced to fail over to the secondary (backup) site. 

The failover and failback process must be executed manually. To do this you need to create Ansible playbooks to map entities between the sites, and to manage the failover and failback processes. The mapping file instructs the Red Hat Virtualization components where to fail over or fail back to on the target site. 

The following diagram describes an active-passive setup where the machine running Red Hat Ansible Engine is highly available, and has access to the oVirt.disaster-recovery Ansible role, configured playbooks, and mapping file. The storage domains that store the virtual machine disks in Site A is replicated. Site B has no virtual machines or attached storage domains. 

##### Active-Passive Configuration

![](https://github.com/jskorzyn/RHV-4.2-LABs/blob/master/files/RHV_Disaster_Recovery_Passive_01.png)


When the environment fails over to Site B, the storage domains are first attached and activated in Site B’s data center, and then the virtual machines are registered. Highly available virtual machines will fail over first. 

##### Failover to Backup Site

![](https://github.com/jskorzyn/RHV-4.2-LABs/blob/master/files/RHV_Disaster_Recovery_Passive_02.png)


You will need to manually fail back to the primary site (Site A) when it is running again. 

### Intro

Show slide with link to survey and ask attendees to answer it.  

_(Open survey on browser)_

Pretend this survey is an important business service that should continue working if the company data center fails. So the company has a disaster recovery data center that can run workloads in case the primary one fails. This means all data must be regularly replicated from the primary to the secondary site.

In this demonstration we will show how this can be achieved using Red Hat Technologies like:

*  RHV;
*  Gluster;
*  Ansible Tower.

First, let’s stop this survey and check results so that we can later validate that same data is available after the workloads are migrated to disaster recovery data center.

_(From survey Settings page, Stop the survey)_

_(From survey Statistics page, show results)_

### Simulate Failure

This is Ansible Tower interface and I’ll use an Ansible playbook to simulate site failure, stopping services and rebooting hypervisors on primary site. Obviously in a real world environment the failure could happen anytime and we would have and the amount of data that would be lost after failover would be determined by business and technology constraints. In our case, I’m forcing a synchronization to ensure latest data is available, while in real world this would be an scheduled task.

### Failover

_Takes 3 minutes since the start of workflow_

Now that we can see that the data center is gone and the application can’t be accessed, let’s run the failover workflow, that includes tasks to prepare storage in disaster recovery site, to configure RHV to use the storage and to make the VMs available again.

This process takes a while as RHV imports the Storage Domain.

On each site, Production and Disaster Recovery, we have a RHV cluster and a Gluster cluster. 
The Gluster volume used by RHV in primary site geo-replicates all data to another volume in Disaster Recovery site. To have better control of data integrity this is typically done at predetermined intervals.

The goal of using Tower to automate the process is that it’s possible to give access to the playbooks only to people that should have control of the Disaster Recovery Process. In our example, it’s dr-admin. Also, Tower includes the possibility to create workflows, which are sequences of tasks linked together, in way which makes it possible to take different paths depending if the tasks fails or succeeds. In our case, we could have included some other set of tasks if “storage failover” had failed.

### Start VM

Now that the VMs were imported and are available in the Disaster Recovery site, we can validate that data was replicated as planned.

_(start vm / access statistics)_

### Failback

Failback isn't yet integrated into Tower as it requires a change in oVirt.disaster-recovery role, as it currently includes a "pause" during the process and it isn't possible to execute the full set of tasks as Tower doesn't allow interactive playbooks like this. This environment will be updated soon to remove this limitation.

However, it's possible to connect to the workstation VM (workstation-$GUID.rhpds.opentlc.com) via SSH with the cloud-user login and execute the following set of tasks:

#### When Primary site is up again

In this demo environment, run the playbook to start services

>     ansible-playbook failure/start.yml

Ensure Gluster isn't georeplicating from primary to secondary and that RHV isn't writing to the Gluster storage domain

>     ansible-playbook playbooks/cleanup.yml --tags "clean_engine"

Ensure Secondary is georeplicating to Primary

>     ansible-playbook playbooks/reverse_georep.yml

#### When Ready to Failback to Primary

Execute the failback playbook - requires shutting down VMs on secondary

>     ansible-playbook playbooks/failback.yml --tags "fail_back"

```
Failover tasks are documented and working properly. 
Failback is working only by executing playbooks directly from workstation as there's an issue with 
oVirt.disaster-recovery role which makes it incompatible with Ansible Tower.
```
