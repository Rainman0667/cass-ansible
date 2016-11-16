## Introduction
This is an [Ansible Playbook](http://docs.ansible.com/playbooks.html) which has been influenced by Joel Jacobson's [DSE Deployer](https://github.com/joeljacobson/dse-deployer). With this playbook you can install, configure and upgrade multi-datacenter clusters running any version of [DataStax Enterprise](http://www.datastax.com/what-we-offer/products-services/datastax-enterprise) greater than 4.8.9. The playbook also supports the provisioning of DSE Analytics and Search. The playbook attempts to follow Ansible best practices and is configurable via group and host variables. In addition to DSE, the playbook can also deploy and configure [OpsCenter](http://www.datastax.com/products/datastax-enterprise-visual-admin)on a separate server and install and configure DataStax agents.

## Instructions
Clone the Playbook:
```
git clone https://github.com/gregory-smith/cass-ansible.git
```
Make sure you have SSH access to your hosts.

Have [Ansible](http://docs.ansible.com/intro_installation.html) installed on your machine. Requires Ansible 2.0 or higher.

Edit the ```hosts``` inventory file and add opscenter, seeds and non-seed sections.

To build a DSE cluster:

Run ```ansible-playbook -i inventory/hosts site.yml```

The playbook has the ability to download and 'stage' multiple versions of DSE via the variable 'stage_versions'. Once the DSE software has been staged, the current active version of the software can configured via the 'active_version' variable. This allows for very rapid upgrades of clusters using the playbook, by simply modifying the current 'active_version'.

```
dse:
  stage_versions: ["4.8.9","4.8.10","4.8.11","5.0.3"]
  active_version: "5.0.3"
```

### Enabling DSE Search and Analytics

To enable Spark on a node simply modify the inventory file :

```1.2.3.4 dc=Data1 rack=rack1 dse_spark=1```

To enable Solr on a node simply modify the inventory file :

```1.2.3.4 dse_solr=1```


### multi-dc deployments

Edit the ```hosts``` inventory file and simply specify the DC and Rack for the hosts. The inventory below will build a cluster with two DC's Data1 and Data2 with DSE Search enabled in Data1.

````
[seeds]
192.168.0.1 dc=Data1 rack=rack1 dse_search=1
192.168.10.1 dc=Data2 rack=rack1

[non-seeds]
192.168.0.2 dc=Data1 rack=rack2 dse_search=1
192.168.10.2 dc=Data2 rack=rack2
````

### Disk configuration
This playbook will configure DSE to store data, commitlogs and saved_caches under a directory called /db which will be owned by the user 'cassandra'. If the variable ````configure_disks```` is set to true and the parameter ````db_disks```` contains a comma separated list of block devices. Then this playbook will group the devices into a single volume group, create a logical volume and then mount a filesystem at ````/db````.   

````
configure_disks: false
db_disks: "/dev/xvdb,/dev/xvdc1,/dev/xvdc2,/dev/xvdc3"
````   

###  OpsCenter
This playbook will enable authentication to the OpsCenter UI, for details of the default user and password refer to the OpsCenter documentation above. This setting is controlled by the following variable:

````
opscenter:
  authentication: true
````

The playbook will also correctly configure the cluster definition within OpsCenter and you should see your cluster in OpsCenter when you log in.

Currently the playbook, will allow you to deploy a cluster, supporting muultiple DC's and monitor this via OpsCenter. In addition rolling upgrades are possible by simply modifying the 'stage_versions' and 'active_version' variables. This is a work in progress, my plan is to slowly improve on this, I have a long list of things I would like to add.
