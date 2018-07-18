osp-cloud-admin
============================================

The purpose of this role is to define and manage any global/cloud administrator-controlled 
OpenStack resources (i.e. domains, project admins, domain admins, flavors, et cetera)
within an OpenStack Platform installation utilizing either Keystone v2 or v3.

Requirements
------------

- Ansible 2.4+
- Python Shade library 1.9+
- python-openstackclient

Role Variables
--------------

/path/to/your/playbooks/host_vars/localhost:

    ---
    osp_cloud_admin:
      roles:
        - "admin"
        - "_member_"
      global_public_network:
        cidr: 10.16.192.0/20
        provider_network_type: vlan
        provider_physical_network: datacentre
        provider_segmentation_id: 2192
        dns_nameservers: "10.16.1.10,10.16.1.11"
        gateway_ip: 10.16.207.254
        allocation_pool_start: 10.16.192.111
        allocation_pool_end: 10.16.207.250
      domains:
        - name: "default"
          description: "default domain"
          admins:
            - name: admin
        - name: "BADGER"
          description: "BADGER domain"
          admins:
            - name: asmuckettly
    ...

Role Dependencies
------------

None

Example Playbook
----------------

    ---
    - hosts: localhost
      connection: local
      become: false
      gather_facts: false
    
      roles:
        - osp-cloud-admin
    ...


Notes
-----
You may have already noticed that there are no group_vars required for this role; however, there is a requirement for all of the authentication credentials to be sourced in the same shell that is used to execute the playbook. Prior to running the playbook, you should be able to run `env | grep "OS_"` and see all the environment variables you'll need defined. Then, from that same terminal prompt, you will execute the playbook. Trying to execute the `source openrc` (or similar) command inside of the playbook itself does not work, because each task inside of the playbook is a child process of the shell used to execute the playbook. So, once each task completes, any env vars that it set that were specific to that child process, are dumped. These do not flow back up to the parent process that launched it.

The whole reason these variables are even required, is because the `os_` series of Ansible modules requires authentication credentials in order to auth to the OpenStack installation it is managing. The modules are relatively uniform in their authentication options, and each of them offers three different ways to authenticate. The first is `cloud-client-config`. This python library reads some variables from a location on the filesystem (a cloud.yml) file. The file is setup to define multiple clouds in a single place and call them by a nickname. The modules use the `cloud` parameter to reference this nickname, and look for these files in specific directories (e.g. /etc/openstack/). This method appears to have some limitations related to how it defines and stores some of the Keystone/Identity v3 parameters though. 

A similar authentication method used by these modules is the `auth` series of parameters that can be defined. These, by and large, have a one-to-one equivalency to the bash env vars defined when you `source openrc` the OpenStack provide credentials that you can download from Horizon. There are; however, some exceptions to that, which make it impractical for use. 

Additionally, It is extremely advantageous to use the same exact authentication method that the `python-openstackclient` uses when it authenticates to Keystone to perform API calls. As a result, sourcing an OpenStack provided `openrc` file is the authentication method of choice for this role. You'll notice a test of the env vars is performed at the front end of the playbook, and you are afforded the opportunity to review the env vars before the playbook proceeds with its run. 

The sourced environment variables are displayed when it reaches the appropriate step in the playbook. You can skip the review process by hitting <CTRL>+<C> and then <C> (Continue) or you can abort by hitting <CTRL>+<C> and then <A> (Abort) when prompted.


Usage
------- 
Executing the playbook is relatively simple. Since it is executed on the localhost, there is no need for an inventory file. Ansible will throw a little warning, but its default behavior is to execute on localhost when you don't provide a user-defined group for it to run the playbook against. You'll just need to ensure you've sourced your OpenStack credentials file prior to running the playbook. Here is an example of the commands:


```
$ pip freeze 2>&1 | egrep -i "shade"   # Ensure the "shade" module is listed in the output
$ source ./openrc    # Or whatever the name of your global-admin credentials file is...
$ ansible-playbook osp-cloud-admin.yml
```


License
-------

MIT

Author Information
------------------

The Development Range Engineering, Architecture, and Modernization (DREAM) Team.
