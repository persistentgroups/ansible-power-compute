## Ansible playbooks for deploying Openstack Compute
The ansible playbooks are used to add a compute node to an existing Openstack control plane.Currently tested on x86 and Power(PowerKVM/UbuntuKVM).

### Required Environment
* Openstack control plane
* Deployer node for running ansible playbooks
* Target node for compute installation

Below steps are performed on deployer node to install compute on target(x86/Power)
## 1. prerequisites
#### 1.1. Get the source code
#### 1.2. Install ansible and its dependencies

```
apt-get install python-pip python-dev
cd <source-code-folder>
pip install -r requirements.txt
```
#### 1.3. Add target node ips in /etc/hosts
```
<compute_ip> compute1
<controller_ip> controller1
```
#### 1.4. Configure Ansible playbooks
```
cd <source-code-folder>
```
Provide corresponding hosts in `envs/setup/compute/hosts`

Example: 
```
# cat envs/setup/compute/hosts
[controller]
controller1
[db_arbiter]
compute1
[mongo_arbiter]
compute1
[compute]
compute1
```
Provide compute node configurations in `envs/setup/compute/group_vars/all.yml` 
```
  | Parameter                       |  Description                            |  
  | --------------------------------|-----------------------------------------|
  |   fqdn: controller              |  Domain name of the controller          |
  |   undercloud_floating_ip        |  Controller IP                          |
  |   undercloud_cidr               |  cidr from which cloud can be accessed  |
  |   primary_interface_controller  | Ethernet port,to which controller node  |
  |                                 | management IP got assigned.             |
  |                                 |  (example:ansible_<controller_ethport>) |
  |  primary_interface_compute      | Refer interface name on compute node    |
  |  openstack:                     |                                         |
  |     controller_ips:             |    Controller IPs                       |
  |         - <controller-ip>       |                                         |
  |  admin_tenant_name              |   Tenant to which nova user is added.   |
                                    |   Refer below parameter in nova.conf of |
                                        control plane.
  |   endpoints.auth_uri            | Refer keystone endpoints and override the    
      http://{{ fqdn }}:35357        corresponding urls. 
  |                                 |     (default values are provided at    
                                       roles/endpoints/defaults/main.yml ). 
  |                                 |
    
  |  identity_api_version           |  keystone identity api version          |
                                                                              |
  |  Secrets:                       |      Refer nova.conf and neutron.conf for     
  |                                         setting values for secrets section|   
  |      nova_db_password           | 
  |      nova_service_password      |                                         |
  |      rabbit_password            |                                         |
  |      admin_password             |                                         |
  |      metadata_proxy_shared_secret|                                        |
  |      neutron_db_password        |
  |      neutron_service_password   |                                         |
  |  neutron:                       |                                
  |      source:                                                              |
            rev: 'stable/kilo'      |       Openstack release                 |
  |      plugin                     |       Neutron core plugin               |
  |      mech_driver                |       Networking mechanism driver (Allowed                                                                                  |
                                            values are linuxbridge, ovs)      |
                                            ovs for Openvswitch plugin        |

  |      tenant_network_type        |  List of network_types to allocate as   
                                      tenant networks. The default value 'local' 
                                      is useful for single-box testing but 
                                      provides no connectivity between hosts. 
                                      (Allowed values are vlan,gre,vxlan,local)
  |      tunnel_types                |  Network tunnel types (allowed values are                                
                                        gre,vxlan)                           |
  |     lbaas:                       |  Enabling/disabling lbaas             |
         enable: False	
  |    l2_population:                |  Add l2_population to mech drivers list|
          enable : False                                                      |
  |      l3ha:                       | High availability for virtual routers  |
            enabled: False                                                    |
  |  network_vlan_ranges: physnet:   | Ranges of VLAN tags on each            |
                                     | physical_network available for allocation                                                                                
                                       as tenant networks.
  |  ceph.enabled                    |  Enable/Disable the ceph configuration on                                                                                
                                        compute to enable ceph as nova backend
  |   nova.backend_ceph              |  configure nova with ceph backend.     |
  |  libvirt_type: kvm               |     libvirt domain type                |
  |  ssl.crt                         |   SSL certificate used at controller   |
```
#### 2. Enable ssh login without password
Ref#: http://www.tecmint.com/ssh-passwordless-login-using-ssh-keygen-in-5-easy-steps

#### 3. Start the deployment
`ansible-playbook -i envs/setup/compute/hosts compute.yml`

