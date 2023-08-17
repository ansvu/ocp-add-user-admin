## Role Description

The purpose of the new role, ocp-disable-console, is to disable and remove the Console Gui functionality from the OpenShift cluster. However, it will ensure that the console CR remains intact(forbidden to remove), while routes, services, pods, and accessibility related to the Console Gui are disabled and deleted. It is important to note that elements such as ClusterOperator and ClusterVersion will still be present. The role includes a check to verify if the Route Info has already been removed and if the Console function is disabled. In such cases, it will skip the remaining tasks, avoiding unnecessary actions.

Note: This role have been tested with `SNO` and `MMO` clusters.

For examples,
```shellSession
$ oc get clusterversion version -o jsonpath='{.spec.capabilities}{"\n"}{.status.capabilities}{"\n"}'
{"enabledCapabilities":["CSISnapshot","Console","Insights","Storage","baremetal","marketplace","openshift-samples"],"knownCapabilities":["CSISnapshot","Console","Insights","Storage","baremetal","marketplace","openshift-samples"]}

$ oc get console cluster -o json |jq -r '.kind'
Console
```

## Requirements

Any pre-requisites are the kubeconfig to access OCP or export KUBECONFIG manually so it depends to pass kubeconfig inside tasks or not.  
Check and update plays/disable_openshift_console.yml:
```yaml
---
- name: "Disable OpenShift Console Gui"
  hosts: localhost
  environment:
    KUBECONFIG: '/tmp/kubeconfig'
  roles:
    - name: ocp-disable-console
```
- Update ansible.cfg to include roles directory
```shellSession
$ cat ~/.ansible.cfg 
[defaults]
roles_path  = $PWD/roles
```

## Role Variables

Default parameter e.g. openshift_console_ns is set to default in defaults/main.yml.  
Or if console namespace is changed then define the disable console inventory file.  
remove_openshift_console_inv.yml:  
```yaml
all:
  vars:
    openshift_console_ns: openshift-console
```

## Dependencies

None.

## How to run the Playbook

```shellSession
$ cat remove_openshift_console.yml 
- import_playbook: plays/disable_openshift_console.yml

$ cat remove_openshift_console_inv.yml 
all:
  vars:
    openshift_console_ns: openshift-console

$ tree roles/ocp-disable-console/
roles/ocp-disable-console/
├── defaults
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
└── templates
    └── disable_openshift_console.yaml

$ tree plays
plays
├── create_user_admin.yml
└── disable_openshift_console.yml
```

## How to play the ansible playbook?
```shellSession
$ ansible-playbook -i remove_openshift_console_inv.yml  remove_openshift_console.yml -v
```
```yaml
TASK [ocp-disable-console : Get Openshift Console Route Information] *****************************************************************************************************************
ok: [localhost]

TASK [ocp-disable-console : Display route information] *******************************************************************************************************************************
ok: [localhost] => {
    "route_info": {
        "api_found": true,
        "changed": false,
        "failed": false,
        "resources": []
    }
}

TASK [ocp-disable-console : Start Disable Openshift Console Gui by PATCH Console CR] *************************************************************************************************
skipping: [localhost]

TASK [ocp-disable-console : Display Console result after it got removed] *************************************************************************************************************
skipping: [localhost]

TASK [ocp-disable-console : Check if console pods are removed from openshift-console] ************************************************************************************************
skipping: [localhost]

TASK [ocp-disable-console : Display pods list information] ***************************************************************************************************************************
skipping: [localhost]

TASK [ocp-disable-console : Display Console GUI is already disabled Message] *********************************************************************************************************
ok: [localhost] => {
    "msg": "Console GUI is already disabled"
}

PLAY RECAP ***************************************************************************************************************************************************************************
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0   
```

## License

BSD

## Author Information
Andrew Vu avu@redhat.com
