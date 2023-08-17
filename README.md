## Role Description

This versatile Ansible role, named `ocp-create-user-admin`, is designed primarily to facilitate the addition of a new admin user to OpenShift, regardless of whether it's a Single Node OpenShift (SNO) or a Multi-Node Managed OpenShift (MMO) cluster. In cases where existing users are already present, the role will `automatically retrieve` their usernames and passwords from the existing secret. Subsequently, it merges this data with the details of the new admin user and updates the secret correspondingly. Additionally, if the httpd-tools RPM package is not installed, the role will handle its installation automatically. Furthermore, the role comprehensively automates the process of generating htpasswd entries, both for entirely new users and when combining new and existing user/password pairs. Moreover, the role includes the capability to identify the presence of an existing secret in the OAuth configuration, and it will proceed to extract the relevant Identity Provider information as well.

Upon execution of the OAuth tasks, the OAuth pod in the "openshift-authentication" namespace will recycle the changes by deleting the pod and recreating it with the updated configuration.

In addition to user management, the role efficiently creates clusterrolebindings for each user listed in the users_list, effectively granting them appropriate access rights as required.

Furthermore, if the user defines the `k8s_auth_api_host` variable, then the role will take the initiative to check and test the newly created admin user's login accessibility against the provided Kubernetes API host.

With its comprehensive functionality, this Ansible role streamlines the process of adding and managing admin users in OpenShift cluster, ensuring a secure and efficient administration process.

**Note:** If new user admin is same as current login `oc whoami`, it won't do nothing. 

## Requirements
- Update ansible.cfg to include roles directory
```shellSession
$ cat ~/.ansible.cfg 
[defaults]
roles_path  = $PWD/roles
```
- An OCP kubeconfig to access cluster
- OpenShift API Authentication Host 
  If user wants to test newly created accessibility, then define and update `k8s_auth_api_host` parameter from add_user_admin_inv.yml file accordingly. If not, then comment out this parameter e.g `#k8s_auth_api_host`  
  Example,  
  https://api.{{cluster_name}}.{{domain_name}}:6443  
  This information can be retrieved with this cmd `oc whoami --show-console`
```yaml
  add_user_admin_inv.yml:  
  k8s_auth_api_host: "https://api.nokiavf.hubcluster-1.lab.eng.cert.redhat.com:6443"
```
- Check and update plays/create_user_admin.yml file
```yaml
---
- name: "Create OpenShift User Admin"
  hosts: localhost
  environment:
    KUBECONFIG: '/tmp/kubeconfig'  
  roles:
    - name: ocp-create-user-admin
```
- Update and Add User Admin Inventory File 
```yaml
---
all:
  vars:
    #These namespaces are optional to define here since it sets defaults/main.yml
    openshift_config_ns: openshift-config
    openshift_auth_ns: openshift-authentication
    
    # only one admin user supports 
    admin_user: adminuser
    admin_passwd: Adminuser123
    os_config_dir: /tmp

    # k8s_auth_api_host is optional and it should be comment by default
    k8s_auth_api_host: "https://api.nokiavf.hubcluster-1.lab.eng.cert.redhat.com:6443"
```

## Role Variables
Name                     | Default                                                                    | Description
-------------------      | ------------                                                               | -------------
openshift_config_ns      | openshift-config                                                           | This namespace uses to create htpasswd secret for OpenShift configuration.
openshift_auth_ns        | openshift-authentication                                                   | This namespace uses to check Oauth PODs status when new htpasswd secret are updated.
admin_user               | None                                                                       | Mandatory. Define new admin user that need to add to the OpenShift cluster
admin_passwd             | None                                                                       | Mandatory. Set new admin user password
os_config_dir            | None                                                                       | Mandatory. Define this directory where users.htpasswd will be used to store newly generated htpasswd e.g. new or append


## Dependencies

None.


## How to run this Playbook
```yaml
$ cat plays/create_user_admin.yml 
---
- name: "Create OpenShift User Admin"
  hosts: localhost
  environment:
    KUBECONFIG: '/tmp/kubeconfig'  
  roles:
    - name: ocp-create-user-admin

$ cat add_user_admin_inv.yml 
---
all:
  vars:
    #These namespaces are optional to define here since it sets defaults/main.yml
    openshift_config_ns: openshift-config
    openshift_auth_ns: openshift-authentication

    # only one admin user supports
    admin_user: adminuser
    admin_passwd: Adminuser123
    os_config_dir: /tmp

    # k8s_auth_api_host is optional and it should be comment by default
    # k8s_auth_api_host: "https://api.nokiavf.hubcluster-1.lab.eng.cert.redhat.com:6443"

$ tree .
.
├── add_user_admin_inv.yml
├── plays
│   └── create_user_admin.yml
├── roles
    └── -ocp-create-user-admin
        ├── defaults
        │   └── main.yml
        ├── README.md
        ├── tasks
        │   ├── check_save_idp_secret_name_if_exists.yml
        │   ├── extract_ocp_config_secret_htpasswd.yml
        │   ├── generate_htpasswd_and_encoded.yml
        │   ├── main.yml
        │   ├── start_create_user_admin.yml
        │   └── verify_user_admin_access.yml
        └── templates
            ├── admin_user_oauth.yaml
            ├── create_admin_user_clusterrolebinding.yaml
            └── create_admin_user_secret.yaml

```
- Run Playbook with Testing newly create admin user  
Note: this parameter `k8s_auth_api_host` in add_user_admin_inv.yml must be commented it out  
`k8s_auth_api_host: "https://api.nokiavf.hubcluster-1.lab.eng.cert.redhat.com:6443"`

```shellSession
$ ansible-playbook -i create_user_admin_inv.yml add_user_admin.yml
```
- Show the output when user checking  
```yaml
TASK [ocp-create-user-admin : Get current OpenShift login user] 
TASK [ocp-create-user-admin : Set the OpenShift user] 
TASK [ocp-create-user-admin : Retrieve OpenShift Identity Provider and Secret Name from Oauth] 
TASK [ocp-create-user-admin : Check if directory exists] 
TASK [ocp-create-user-admin : Fail if the configs directory does not exist] 
TASK [ocp-create-user-admin : Get OAuth cluster configuration for IDP and Secret Name] 
TASK [ocp-create-user-admin : Set IDP Name for later use if it exists otherwise use default idp name] 
TASK [ocp-create-user-admin : Print Identity Provider Name] 
TASK [ocp-create-user-admin : Set fact to check if secret exists in oauth config] 
TASK [ocp-create-user-admin : Set Secret Name for later use if it exists otherwise use default secret name] 
TASK [ocp-create-user-admin : Set this flag if existing secret is present] 
TASK [ocp-create-user-admin : Print Secret Name] 
TASK [ocp-create-user-admin : Print Existed Secret Flag] 
TASK [ocp-create-user-admin : Extract and save existing user and password of htpasswd to a file] 
TASK [ocp-create-user-admin : Extract OpenShift Config Secret to get exist user from htpasswd data] 
TASK [ocp-create-user-admin : Print Secret Data] 
TASK [ocp-create-user-admin : Decode htpasswd value] 
TASK [ocp-create-user-admin : Print decoded htpasswd value] 
TASK [ocp-create-user-admin : Save existing htpasswd user and password to /tmp/users.htpasswd] 
TASK [ocp-create-user-admin : Generate htpasswd user admin] 
TASK [ocp-create-user-admin : Install httpd-tools package] 
TASK [ocp-create-user-admin : Display whether httpd-tools is installed or not] 
TASK [ocp-create-user-admin : Check if users.htpasswd file exists in /tmp] 
TASK [ocp-create-user-admin : Generate htpasswd entry and append or create users.htpasswd] 
TASK [ocp-create-user-admin : Print htpasswd file after generation] 
TASK [ocp-create-user-admin : Encoding the htpasswd file from /tmp/users.htpasswd] 
TASK [ocp-create-user-admin : Set htpasswd encoded for later use in Oauth] 
TASK [ocp-create-user-admin : Print htpasswd encoded password content] 
TASK [ocp-create-user-admin : Print htpasswd encoded data] 
TASK [ocp-create-user-admin : Start Creating OpenShift User Admin] 
TASK [ocp-create-user-admin : Get Oauth Pods Before Update Oauth Secret] 
TASK [ocp-create-user-admin : Set Fact for Pod Ready Condition before] 
TASK [ocp-create-user-admin : Print htpasswd encoded data] 
TASK [ocp-create-user-admin : Create admin user secret] 
TASK [ocp-create-user-admin : Update admin user oauth] 
TASK [ocp-create-user-admin : Wait for Oauth Pods Restart Completion] 
TASK [ocp-create-user-admin : Check Oauth PODs and wait until all are running] 
TASK [ocp-create-user-admin : Print PODs Ready Condition before and after] 
TASK [ocp-create-user-admin : Create admin user clusterrolebinding] 
TASK [ocp-create-user-admin : Testing user admin access for adminuser] 
TASK [ocp-create-user-admin : Verify accessibility of admin user for adminuser] 
TASK [ocp-create-user-admin : Print OC login result]
task path: /root/nokia/nokia-automation/roles/test-ocp-create-user-admin/tasks/verify_user_admin_access.yml:8
ok: [localhost] => {
    "oc_login_result": {
        "changed": true,
        "cmd": "oc login -u \"adminuser\" -p \"Adminuser123\" \"https://api.abi.hubcluster-1.lab.eng.cert.redhat.com:6443\" --insecure-skip-tls-verify",
        "delta": "0:00:00.219674",
        "end": "2023-08-16 22:33:09.095688",
        "failed": false,
        "failed_when_result": false,
        "msg": "",
        "rc": 0,
        "start": "2023-08-16 22:33:08.876014",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "WARNING: Using insecure TLS client config. Setting this option is not supported!\n\nLogin successful.\n\nYou have one project on this server: \"default\"\n\nUsing project \"default\".",
        "stdout_lines": [
            "WARNING: Using insecure TLS client config. Setting this option is not supported!",
            "",
            "Login successful.",
            "",
            "You have one project on this server: \"default\"",
            "",
            "Using project \"default\"."
        ]
    }
}
META: role_complete for localhost
META: ran handlers
META: ran handlers

PLAY RECAP ***************************************************************************************************************************************************************************
localhost                  : ok=42   changed=5    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

## Limitation

For now, user needs to generate new admin user using htpasswd tool, and get/extract the existings `secret` and `identity-provider` names. Next step is to automate these requirements.


## License

BSD

## Author Information

Andrew Vu: avu@redhat.com
