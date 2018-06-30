# vagrant-origin-multimaster


## Building a test Openshift Origin cluster in Vagrant

This objective is to build an Openshift Origin 3.9 cluster in an environment that is self contained for training purposes, it should be 100% disposable and should not be accessible from the Corporate network.

#### Dependencies

* Ansible 2.5.0 or higher
* A clone of https://github.com/openshift/openshift-ansible.git
* Vagrant & VirtualBox

### Remove any existing vagrant hosts

On host upmgt020

```
sh /var/vagrant/jgr78/OSO/provisioning/scripts/remove-atomic-hosts.sh
```

### Create new vagrant hosts

```
sh /var/vagrant/jgr78/OSO/provisioning/scripts/create-atomic-hosts.sh
```

This will take a while, please be patient..........  
It will result in the following hosts being provisioned

| Host Name                      | IP Address      | Description                                    |
|--------------------------------|-----------------| -----------------------------------------------|
| master1.192.168.205.201.xip.io | 192.168.205.201 | First master node of a multi master pair       |
| master2.192.168.205.201.xip.io | 192.168.205.202 | Second master node of a multi master pair      |
| etcd1.192.168.205.201.xip.io   | 192.168.205.203 | Node 1 of the ETCD cluster                     |
| etcd2.192.168.205.201.xip.io   | 192.168.205.204 | Node 2 of the ETCD cluster                     |
| etcd3.192.168.205.201.xip.io   | 192.168.205.205 | Node 3 of the ETCD cluster                     |
| infra1.192.168.205.201.xip.io  | 192.168.205.206 | First Infra node for running cluster services  |
| infra2.192.168.205.201.xip.io  | 192.168.205.207 | Second Infra node for running cluster services |
| infra3.192.168.205.201.xip.io  | 192.168.205.208 | Third Infra node for running cluster services  |

Running a vagrant status will show you all of the hosts that have been created
```
vagrant status
```

### Add additional storage for docker images

Login to the ansible control VM

```
cd /var/vagrant/jgr78/OSO
vagrant ssh oso-ansible
```

Run the add-docker-storage.sh script
```
sh oso-scripts/add-docker-storage.sh
```

### Clone the openshift-ansible repo from Github
```
cd /apps/ansible/origin
rm -rf openshift-ansible
git clone https://github.com/openshift/openshift-ansible.git
cd openshift-ansible
git fetch origin
git checkout -b release-3.9 origin/release-3.9
cd ..
```

### Run the ansible playbooks
```
ansible-playbook -i inventory/vagrant-origin/hosts playbooks/ocp-install.yml -kv 
(password is vagrant)
```

The playbooks will take around 45 minutes to run, once they have finished add the cluster administrator users to both master nodes

```
vagrant ssh oso-master1
sudo echo "jgr78:$apr1$v//LoDln$4aUFYC58yP5zoa.1n75JA/" >> /etc/origin/master/htpasswd
sudo oc adm policy add-cluster-role-to-user cluster-admin jgr78
```

```
vagrant ssh oso-master2
sudo echo "jgr78:$apr1$v//LoDln$4aUFYC58yP5zoa.1n75JA/" >> /etc/origin/master/htpasswd
sudo oc adm policy add-cluster-role-to-user cluster-admin jgr78
```

### Login to the cluster
```
oc login -u jgr78 https://admin.ocp.192.168.205.240.xip.io:8443
Password: **********
Login successful.
```

Make sure the cluster services are running

```
oc get deploymentconfigs
```

```
NAME                                 REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfigs/docker-registry    1          1         1         config
deploymentconfigs/registry-console   1          1         1         config
deploymentconfigs/router             1          3         3         config
```

```
oc get pods
```

```
NAME                       READY     STATUS    RESTARTS   AGE
docker-registry-1-pmj8r    1/1       Running   0          18m
registry-console-1-6bt6l   1/1       Running   0          16m
router-1-7q9ck             1/1       Running   0          19m
router-1-wvnxt             1/1       Running   0          19m
router-1-zw8bk             1/1       Running   0          19m
```
