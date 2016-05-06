### Power Compute Deployment Documentation
This document provides the steps to deploy compute on x86/Ubuntu KVM/PowerKVM. 
In case of any errors or issues during deployment process, you can refer “Help" section.

##### Required Environment
* Openstack control plane
* Deployer node for running ansible playbooks
* Target node for compute installation

#####  Prerequisites 
Below steps are performed on deployer node to install compute on target (x86/ Power (UbuntuKVM/PowerKVM))

1. Get the source code
    ```
    git clone https://github.com/persistentgroups/ansible-power-compute
    ```
2. Install ansible and its dependencies
    ```
    apt-get install python-pip python-dev
    cd ansible-power-compute
    pip install -r requirements.txt
    ```
3. Add target node ips in /etc/hosts
    ``` 
    <compute_ip> compute1
    <controller_ip> controller1
    ```

4.	Configure ansible playbooks
    ```
    cd ansible-power-compute
    ```

    Provide corresponding hosts in `envs/setup/compute/hosts`
	      
        ```	   
        Example: 
        [controller]
        controller1
        [db_arbiter]
        compute1
        [compute]
        compute1
        ```
5. Provide compute node configurations in envs/setup/compute/group_vars/all.yml
#### Enable ssh login without password
This will allow ssh login to compute node without password through ansible scripts

  Reference:
    http://www.tecmint.com/ssh-passwordless-login-using-ssh-keygen-in-5-easy-steps/
#### Deployment
``` 
Command: # ansible-playbook -i envs/setup/compute/hosts compute.yml
```

#### Testing
On compute node check the service status
```
service nova-compute status 
service neutron-linuxbridge-agent status
or
service neutron-openvswitch-agent status
```
On controller node check the hosts and agent list
```
nova host-list
neutron agent-list
```
#### Help!
This section provides details of inconsistent errors and workarounds.
```
Error:
TASK: [openstack-source | pip install project_name] ***************************
failed: [powerkvm] => {"attempts": 3, "cmd": "/opt/ansible/openstack-11.0-master/nova/bin/pip install /opt/stack/nova", "failed": true}
msg: Task failed as maximum retries was encountered	

Solution:
Upgrade pip on compute(target) node:

Command: /opt/ansible/openstack-11.0-master/nova/bin/pip install --upgrade pip
```
```
Error:
pkg_resources.VersionConflict: (setuptools 0.9.8 (/opt/ansible/openstack-11.0-master/neutron/lib/python2.7/site-packages), Requirement.parse('setuptools>=17.1'))
Cleaning up...
Command python setup.py egg_info failed with error code 1 in /opt/ansible/openstack-11.0-master/neutron/build/funcsigs

Solution:
Run below command on compute(target) node
pip install –upgrade pip
```

```
Error:
python-barbicanclient requires pbr>=1.8 bit neutron and neutron-lbaas requirement has pbr!=0.7,<1.0,>=0.6 

Solution:	
Upgrade pbr and install barbicanclient on compute node
pip install –upgrade pbr
pip install python-barbicanclient 
```
```
Error:
Build of instance 4eea2ade-300d-4abd-a995-7d0a7535c9a5 was re-scheduled: internal error: early end of file from monitor: possible problem: 
 Error: Cirrus VGA not available     

Solution: 
glance image-update <image-id> --property hw_video_model=vga
```

```
Error:
SSL ERROR

Solution:
client.self_signed_cert: false in roles/client/defaults (this parameter is used in stackrc)
Need to add ssl.crt and client.self_signed_cert: true 
parameter  in all.yml  if ssl certs are used
```


