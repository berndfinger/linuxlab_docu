## Contents

 - [1. Overview](#overview)
 - [2. Installation](#installation)
 - [3. New features in version 3](#New-features-v3)
    + [3.1.New Features added to Support Cluster Installation](#31-cluster-installation)
 - [4. Known Issues](#Known-issues)
    + [4.1. Roles produce limited output when running in check mode](#41-Roles-produce)
    + [4.2. Extended check (=assert) parameters are not recognized in previous versions of the roles](#42-extended-check)
    + [4.3. Role `sap-preconfigure` fails if DNS domain is not set on the managed node](#43-Role-sap-preconfigure-fails)
    + [4.4. The assertion for getting the current status of the CPU Governor for performance (x86_64 platform only) fails](#44-assertion)
 - [5. Quick Start](#Quick-Start)
    + [5.1. Prepare the Control Node](#51-prepare)
    + [5.2. Configure the local system](#52-Configure-the-local-system)
    + [5.3. Verify the local system](#53-verify)
    + [5.4. Configure remote systems](#54-Configure-remote-systems)
    + [5.5. Verify a remote system](#55-verify)
 - [6. Detailed Description](#Detailed-Description)
    + [6.1. System Roles and SAP Notes](#61-System-Roles-and-SAP)
    + [6.2. Implemented SAP Notes](#62-Implemented-SAP-Notes)
    + [6.3. Role variables](#63-Role-Variables)
 - [7. Examples](#examples)
    + [7.1. Prepare two systems for SAP HANA](#71-prep-two-systems)
    + [7.2. Configure four systems for SAP NetWeaver](#72-config-four-systems)
    + [7.3. Report the SAP HANA notes compliance status of a RHEL system](#73-report)
    + [7.4. Example of Setting up a 2 Node HANA Scale-Up cluster](#74-2node-hana)
 - [8. Related Information](#Related-Information)

## 1. Overview {#overview}

Red Hat Enterprise Linux (RHEL) 7 [RHEA-2019:3190](https://access.redhat.com/errata/RHEA-2019:3190) introduced RHEL System Roles for SAP to assist with remotely or locally configuring a RHEL system for the installation of SAP HANA or SAP NetWeaver software. RHEL System Roles for SAP development is based on the [Linux System Roles](https://linux-system-roles.github.io/) upstream project.

**RHEL System Roles** is a collection of roles executed by Ansible to assist administrators with server configuration right after the servers have been installed. These roles are provided in the RHEL Extras repository. In contrast, **RHEL System Roles for SAP** is provided in the RHEL for SAP Solutions subscription and can be used by Ansible Engine and Ansible Tower to manage RHEL systems.

The Red Hat Enterprise Linux subscription provides support for RHEL System Roles with [Ansible Engine](https://access.redhat.com/products/red-hat-ansible-engine/), which is available in the Ansible Engine repository (e.g. `ansible-2-for-rhel-8-$(uname -m)-rpms`). However, if you require full support for the [Ansible Engine](https://www.ansible.com/ansible-engine-pricing) itself, a separate [Red Hat Ansible Automation Subscription](https://access.redhat.com/articles/3076221) is necessary. Additional information is available at [Top Support Policies for Red Hat Ansible Automation](https://access.redhat.com/ansible-top-support-policies).

The following RHEL System Roles for SAP are fully supported on control nodes running RHEL 8.2 and later:
- *sap-preconfigure*
- *sap-netweaver-preconfigure*
- *sap-hana-preconfigure*

Note: Beginning with rhel-system-roles-sap-3.2.0-1.el8_4, the role names have changed as follows:
| Previous role name | New role name |
| ----- | ----- |
| *sap-preconfigure*           | *sap_general_preconfigure*   |
| *sap-netweaver-preconfigure* | *sap_netweaver_preconfigure* |
| *sap-hana-preconfigure*      | *sap_hana_preconfigure*      |

Variables in existing playbooks are still recognized. For more information, including using the preconfigure roles as part of collection redhat.sap_install and the new role sap_hana_install, refer to KB article [6857351](https://access.redhat.com/articles/6857351).

The RHEL System Roles for SAP, just like the [RHEL System Roles](https://access.redhat.com/articles/3050101), are installed and run from a central node referred to as the *control node* (which can be Ansible Tower, Red Hat Satellite, or a RHEL 8 or RHEL 7 host). The control node connects to the local host and/or to one or more remote hosts (called *managed nodes* in the context of Ansible), and performs installation and configuration steps on them. It is recommended that you use the latest major release of RHEL on the control node (RHEL 8) and use the latest version of the roles either from the `rhel-system-roles-sap` RPM or from [Red Hat Automation Hub](https://console.redhat.com/ansible/automation-hub/repo/published/sap). The RHEL System Roles for SAP and Ansible packages do not need to be installed on the systems that are being managed/configured.

See the following table for the support status:

| Control Node | Managed Node | Support Status |
| ----- | ----- | ----- |
| RHEL 8.4 or later | RHEL 8.0 or later | fully supported |
| RHEL 8.4 or later | RHEL 7.6 or later | fully supported |
| RHEL 8.4 or later | RHEL 7.5 or earlier | not supported |
| RHEL 8.3 or earlier | RHEL (any release) | not supported† |

<br>

**† Note**: For *control nodes* running RHEL 7.8, RHEL 7.9, or RHEL 8.1, you can use the previous versions of `rhel-system-roles-sap` which are in Tech Preview support status. Please find the instructions for these versions [here](https://access.redhat.com/sites/default/files/attachments/rhel_system_roles_for_sapv1.pdf).

For *control nodes* running RHEL 8.2 or RHEL 8.3, you can use version 2 of `rhel-system-roles-sap` which is fully supported. Please find the instructions for this version [here](https://access.redhat.com/sites/default/files/attachments/rhel_system_roles_for_sapv2_0.pdf).

See the table below for the supported hardware/virtualization/cloud platforms of the managed node:

| Hardware platform | Bare Metal/Virtualization/ Cloud platform | Support Status |
| ----- | ----- | ----- |
| x86_64 | bare metal, Red Hat Virtualization/libvirt, VMware ESX, [Red Hat Certified Cloud and Service Providers](https://www.redhat.com/en/certified-cloud-and-service-providers) | fully supported |
| ppc64le | PowerVM LPARs | fully supported |
| s390x | zVM guest | fully supported: *sap-preconfigure*, *sap-netweaver-preconfigure* |

<br>

**Note**: The roles are designed to be used right after the initial installation of a managed node. Do not run these roles against a SAP or other production system. The role will enforce a certain configuration on the managed node(s), which might not be intended.
**Note**: Before applying the roles on a managed node, verify that the RHEL release on the managed node is supported by the SAP software version that you are planning to install.

Additional to the **RHEL System Roles for SAP** we have introduced a new repository in 2022 [sap-linuxlab](https://github.com/sap-linuxlab/community.sap_install).
This repository is covering these roles, which might be merged in the future into the **automation hub**.

Rolename|Support status|Supported RHEL Release| Controle Node
---|---|--|----
sap_general_preconfigure | Fully supported | RHEL 7.6 and later, RHEL 8 | RHEL 8
sap_netweaver_preconfigure | Fully supported | RHEL 7.6 and later, RHEL 8 | RHEL 8
sap_hana_preconfigure | Fully supported | RHEL 7.6 and later, RHEL 8 | RHEL 8
sap_hana_install | Technology Preview(*) | RHEL 7.6 and later, RHEL 8 | RHEL 8
sap_ha_install_pacemaker|Technology Preview(*) | RHEL 8.2 and later| RHEL 8
sap_ha_prepare_pacemaker|Technology Preview(*) | RHEL 8.2 and late| RHEL 8
sap_ha_set_hana|Technology Preview(*) | RHEL 8.2 and late| RHEL 8

## 2. Installation {#installation}

Use this procedure to install the Ansible Engine and the RHEL System Roles for SAP.

1) Use subscription-manager to list the available Ansible Engine repositories.
`# `**`subscription-manager refresh`**
`# `**`subscription-manager repos --list | grep ansible`**

2) Permanently enable the Ansible Engine repository and the RHEL for SAP Solutions repository using Red Hat Subscription Manager.
Note: The generic version "2" of the Ansible Engine repository provides the latest release of the 2.X stream but it is also possible to specify a certain minor Ansible Engine version such as 2.9.
`# `**`subscription-manager repos --enable=ansible-2-for-rhel-8-$(uname -m)-rpms --enable=rhel-8-for-$(uname -m)-sap-solutions-rpms`**

3) Install Ansible Engine and RHEL System Roles for SAP:
`# `**`dnf install ansible rhel-system-roles-sap`**

The `rhel-system-roles-sap` package is installed to the following locations where `<role>` is the name of the individual role; for example, `sap-hana-preconfigure`. Each role includes a README file that explains all variables and how to use the role.

Documentation: `/usr/share/doc/rhel-system-roles-sap/<role>`
Ansible Roles: `/usr/share/ansible/roles/<role>`

## 3. New features in version 3 {#New-features-v3}

Version 3.1 has the following new features compared to version 2:

- The three roles now support an assertion run, so they can be used to compare the settings of a managed node to the applicable SAP notes. While Ansible supports setting and verifying any modification made to a managed node by design, it can be useful to report the SAP notes compliance of a SAP system from time to time without modifying the system configuration, for example to ensure that system settings are still in place after a manual modification of system parameters. The roles can either fail at each detected violation, or they can report failures but continue running and finally report the number of failures (if any).

- The roles `sap-preconfigure` and `sap-hana-preconfigure` now support a reboot of the managed node if there have been software installations which require it.

- Role `sap-preconfigure` only remounts file system `/dev/shm` if necessary.

- Role `sap-netweaver-preconfigure` now supports the installation of packages which are required for Adobe Document Services.

- The role `sap-hana-preconfigure` no longer sets the SELinux state. This is already done in role sap-preconfigure.

- Configuring role `sap-hana-preconfigure` for using tuned and/or modifying the boot command line has been simplified.

- Role `sap-hana-preconfigure` now supports activating tuned profile sap-hana and also modifying the boot command line. This provides greater flexibility when setting latency related parameters.

- Role `sap-hana-preconfigure` now supports checking if the RHEL minor release is supported for SAP HANA. This behavior can be overridden so that any RHEL 7.6 or greater managed node can be prepared for SAP HANA.

- Role `sap-hana-preconfigure` now supports setting kernel parameters for NetApp NFS as per [SAP Note 3024346](http://launchpad.support.sap.com/#/notes/3024346).

### 3.1 New Features added to Support Cluster installation {#31-cluster-installation}
The specification of a 2 node HANA cluster was simplified to a single file with a lower number of necessary parameters.

An example will be shown in the examples session.
The roles used to create the cluster are now able to use this parameters.

The following roles are used therefor.

System Role|Description
:---|:---
sap_general_preconfigure|System Preparation for SAP
sap_hana_preconfigure|System Preparation for SAP HANA
sap_hana_install|Installation of SAP HANA Database
sap_ha_install_hana_hsr|Configuration of SAP HANA System Replication
sap_ha_prepare_pacemaker|Authentication and Preparation of Nodes for Cluster Creation
sap_ha_install_pacemaker|Initialization of the Pacemaker Cluster
sap_ha_set_hana|Configuration of SAP HANA Resources for SAP Solutions

The necessary variables looks like:

Variable Name|Description|Example
:---|:---|:---
sap_hana_sid|SAP HANA SYSTEM Indentification|DB1
sap_hana_instance_number|Instance Number| '00'
sap_hana_install_master_password|HANA Password|'my_hana-password'
sap_hana_cluster_name|Name of the pacemaker cluster|cluster1
sap_hana_hacluster_password|hacluster password to authenticate pacemaker cluster nodes|'my_hacluster-password'
sap_hana_cluster_nodes|List of nodes and attributes|example see below
sap_hana_vip1|Virtual IP address of the primary HANA database server| 192.168.1.100

Example of sap_hana_cluster_nodes in the vars: section
```
vars:
  sap_hana_cluster_nodes:
    - node_name: node1
      node_ip: 192.168.1.11
      node_role: primary
      hana_site: DC01

    - node_name: node2
      node_ip: 192.168.1.12
      node_role: secondary
      hana_site: DC02
```

## 4. Known issues {#Known-issues}

### 4.1. Roles produce limited output when running in check mode {#41-Roles-produce}

Running roles in check mode will not show all changes which are performed on a system when running in normal mode, as some Ansible modules have no or just partial support for check mode. For example, tasks will not report the values of kernel parameters. For more information on the Ansible check mode, please refer to [https://docs.ansible.com/ansible/latest/user_guide/playbooks_checkmode.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_checkmode.html).

To overcome this restriction, the sap*preconfigure roles can now run in an extended check (=assert) mode.

### 4.2. Extended check (=assert) parameters are not recognized in previous versions of the roles {#42-extended-check}

The roles can run in an assert mode, in which case they do not modify managed nodes but report the compliance of a node with the applicable SAP notes. When running assert mode playbooks with previous versions (1.x or 2.x) of the roles, assert parameters are ignored, causing the roles to modify the managed nodes instead of only checking them. As roles can also be installed in other than the default locations (e.g. using git), it is recommended that you not only check if version 3 of package `rhel-system-roles-sap` is installed but also that the playbooks you are using are calling the roles in their correct, default locations, which is under `/usr/share/ansible/roles`.
For displaying the role path which is used when running the role so that you can verify if it is corresponding to the installed version, you can use the following procedure:
Run the command (replace `PLAYBOOK.YML` by the actual name of the playbook and `HOSTNAME` by the name of the managed node):
`# ansible-playbook PLAYBOOK.YML -l HOSTNAME --step -vv`
Answer the first question with "N":
`Perform task: TASK: Gathering Facts (N)o/(y)es/(c)ontinue:` **`N`**
and the second question with "y":
`Perform task: TASK: sap-preconfigure : include os specific vars (N)o/(y)es/(c)ontinue:` **`y`**

This will display the absolute path name of file `tasks/main.yml` and then abort the play (because the vars file could not be found).
Example output:
~~~
TASK [sap-preconfigure : include os specific vars]
****************************************************************************
task path: /usr/share/ansible/roles/sap-preconfigure/tasks/main.yml:3
fatal: [HOSTNAME]: FAILED! => {"msg": "No file was found when using first_found. Use errors='ignore' to allow this task to be skipped if no files are found"}
~~~

### 4.3. Role `sap-preconfigure` fails if DNS domain is not set on the managed node {#43-Role-sap-preconfigure-fails}

In case there is no DNS domain set on the managed node, which is typically the case on cloud systems, the role `sap-preconfigure` will fail in task *Verify that the DNS domain is set*. To avoid this, set variable `sap_domain` in file `/usr/share/ansible/roles/sap-preconfigure/defaults/main.yml` or run the `ansible-playbook command` with line parameter
`-e "sap_domain=example.com"` (with the domain name being example.com in this case - please replace it by your domain name).

(sap-preconfigure issue #[32](https://github.com/linux-system-roles/sap-preconfigure/issues/32))

### 4.4. The assertion for getting the current status of the CPU Governor for performance (x86_64 platform only) fails {#44-assertion}

When running role `sap-hana-preconfigure` in assert mode against a x86_64 managed node, it might incorrectly report that the current status of the CPU Governor for performance is not as expected.
(sap-hana-preconfigure issue #[180](https://github.com/linux-system-roles/sap-hana-preconfigure/issues/180))

## 5. Quick Start {#Quick-Start}

Use this procedure to configure or verify one or more systems for the installation of SAP NetWeaver or SAP HANA.

### 5.1. Prepare the Control Node {#51-prepare}

RHEL System Roles for SAP requires that the Ansible control node uses locale `C` or `en_US.UTF-8` to display system messages in English. Run the following command on the local host to check the current setting:
`# locale`
The output should display either `C` or `en_US.UTF-8` in the line starting with `LC_MESSAGES=`. If the `locale` command does not produce the expected output, run the following command on the local host before executing the `ansible-playbook` command:
`# `**`export LC_ALL=C`**
Or
`# `**`export LC_ALL=en_US.UTF-8`**

### 5.2. Configure the local system {#52-Configure-the-local-system}

#### Prepare the local system for the installation of SAP NetWeaver

1) Make sure that there is no production software running on the system. The roles will enforce a certain configuration on the system, which typically is intended only **right after** the installation of RHEL and **before** the **initial** installation of SAP software.

2) In case you would like to preserve the original configuration of the server, perform a backup. Typically, these roles are run right after the installation of RHEL, so a backup should not be necessary.

3) Create a YAML file named `sap-netweaver.yml` with the following content:
~~~
- hosts: localhost
  connection: local
  roles:
    - sap-preconfigure
    - sap-netweaver-preconfigure
~~~
Notes:

- The correct indentation (e.g. 2 spaces in front of `roles:`) is essential.

- Beginning with rhel-system-roles-sap-3.2.0-1.el8_4, the role names have changed as indicated in chapter [1](#overview).

5) Make sure there is at least 20480 MB of swap space configured on the local system.

6) Run the RHEL System Roles `sap-preconfigure` and `sap-netweaver-preconfigure` to prepare the managed nodes for the installation of SAP NetWeaver.

`# `**`ansible-playbook sap-netweaver.yml`**

At the end of the playbook run, the command will report that a reboot is required because role `sap-preconfigure` has changed the SELinux state from `enabled` to `disabled`, according to SAP note [2772999](https://launchpad.support.sap.com/#/notes/2772999).

7) Reboot the managed nodes so that the new SELinux state will become effective. If you set the role variable `sap_preconfigure_reboot_ok` to yes, the role will reboot the server as the last step of its execution.

**Note**: By changing role variable `sap_preconfigure_selinux_state` from the default `disabled` to `permissive` before or at the time of running the playbook, you can have the role `sap-preconfigure` set the SELinux state to `permissive`, which is also allowed for SAP NetWeaver on RHEL 8. See the Examples section in this document for more information on setting role variables.

### 5.3. Verify the local system {#53-verify}

In addition to configuring RHEL systems, the RHEL System Roles for SAP can also be used to verify that RHEL systems are configured correctly.

#### Verify if the local system is configured correctly for the installation of SAP NetWeaver

Use the following procedure to get a report of a server's compliance with applicable SAP notes for SAP NetWeaver:

1) Create a YAML file named `sap-netweaver.yml` with the following content:

~~~
- hosts: localhost
  connection: local
  vars:
    sap_preconfigure_assert: yes
    sap_preconfigure_assert_ignore_errors: yes
    sap_netweaver_preconfigure_assert: yes
    sap_netweaver_preconfigure_assert_ignore_errors: yes
  roles:
    - sap-preconfigure
    - sap-netweaver-preconfigure
~~~
Notes:

- The correct indentation (e.g. 2 spaces in front of `roles:`) is essential.

- Beginning with rhel-system-roles-sap-3.2.0-1.el8_4, the role names have changed as indicated in chapter [1](#overview).

2) Make sure you are using RHEL System Roles for SAP version 3.
Because the extended check (=assert) parameters are not recognized in previous versions of the roles, it is important that you do not run this playbook with a previous version of RHEL System Roles for SAP. Otherwise, the system configuration could unintentionally be modified. Please follow the steps outlined in chapter [4.2](#42-extended-check) to verify that you are using RHEL System Roles for SAP version 3 or later, and that the roles are not called from other locations than the default role paths.

3) Run the following command:

~~~
# ansible-playbook sap-netweaver.yml
~~~

In case you would like to get a more compact output, you can filter the output to just display the essential FAIL or PASS information for each assertion. If you are using a terminal with dark background, replace all occurrences of color code **`[30m`** in the following command sequence by **`[37m`**. Otherwise, the output of some lines will be unreadable due to dark font on dark background.

~~~
# ansible-playbook sap-netweaver.yml | awk '{sub ("    \"msg\": ", "")}
  /TASK/{task_line=$0}
  /fatal:/{fatal_line=$0; nfatal[host]++}
  /...ignoring/{nfatal[host]--; if (nfatal[host]<0) nfatal[host]=0}
  /^[a-z]/&&/: \[/{gsub ("\\[", ""); gsub ("]", ""); gsub (":", ""); host=$2}
  /SAP note/{print "\033[30m[" host"] "$0}
  /FAIL:/{nfail[host]++; print "\033[31m[" host"] "$0}
  /WARN:/{nwarn[host]++; print "\033[33m[" host"] "$0}
  /PASS:/{npass[host]++; print "\033[32m[" host"] "$0}
  /INFO:/{print "\033[34m[" host"] "$0}
  /changed/&&/unreachable/{print "\033[30m[" host"] "$0}
  END{print ("---"); for (var in npass) {printf ("[%s] ", var); if (nfatal[var]>0) {
        printf ("\033[31mFATAL ERROR!!! Playbook might have been aborted!!!\033[30m Last TASK and fatal output:\n"); print task_line, fatal_line
     }
     else printf ("\033[31mFAIL: %d  \033[33mWARN: %d  \033[32mPASS: %d\033[30m\n", nfail[var], nwarn[var], npass[var])}}'
~~~

In case you accidentally ran the above command on a terminal with dark background, you can re-enable the default white font again with the following command:

~~~
# awk 'BEGIN{printf ("\033[37mResetting font color\n")}'
~~~

### 5.4. Configure remote systems {#54-Configure-remote-systems}

#### Prepare the control node and ssh access to all managed nodes {#prepare_the_control_node}

1) Verify that the managed nodes are correctly set up for installing Red Hat software packages from a Red Hat Satellite server or the Red Hat Customer Portal.

2) Make sure that you can log in via the ssh command to all managed nodes from the Ansible control node without using a password. See the man pages for `ssh-copy-id` and `man ssh` if you need more information about this topic.

#### Prepare one or more remote servers (managed nodes) for the Installation of SAP HANA

1) Verify that there is no production software running on any of the managed nodes you want to configure.

2) Make sure that the version of SAP HANA you will be installing is supported for the RHEL major and minor release which is installed on the managed nodes. For information on supported RHEL releases for SAP HANA, see SAP note [2235581](https://launchpad.support.sap.com/#/notes/2235581).

3) In case you would like to preserve the original configuration of any of the servers, perform a backup of the server(s). Typically, these roles are run right after installation, so a backup should not be necessary.

4) Create an inventory file or modify file `/etc/ansible/hosts` so that it contains the name of a group of hosts and each host which you intend to configure (=managed node) in a separate line (example for three hosts in a host group named `sap_hana`):
~~~
[sap_hana]
host01
host02
host03
~~~

5) Use some simple commands to verify that you can log in to all three hosts using ssh without password:
`# `**`ssh host01 uname -a`**
`# `**`ssh host02 hostname`**
`# `**`ssh host03 echo test`**

6) Create a YAML file named `sap-hana.yml` with the following content:
~~~
- hosts: sap_hana
  roles:
    - sap-preconfigure
    - sap-hana-preconfigure
~~~
Notes:

- The correct indentation (e.g. 2 spaces in front of `roles:`) is essential.

- Beginning with rhel-system-roles-sap-3.2.0-1.el8_4, the role names have changed as indicated in chapter [1](#overview).

7) Run the RHEL System Roles `sap-preconfigure` and `sap-hana-preconfigure` to prepare the managed nodes for the installation of SAP HANA.
**Note**: Do not run these roles against an SAP or other production system. The role will enforce a certain configuration on the managed node(s), which typically is intended only **right after** the installation of RHEL and **before** the **initial** installation of SAP software.

`# `**`ansible-playbook sap-hana.yml`**

At the end of the playbook run, the command will report for each managed node that a reboot is required, for example because role *sap-preconfigure* has changed the SELinux state from `enabled` to `disabled` (as per requirement in SAP notes [2292690](https://launchpad.support.sap.com/#/notes/2292690) or  [2777782](https://launchpad.support.sap.com/#/notes/2777782)).

8) Reboot the managed nodes so that the new SELinux state will become effective.

### 5.5. Verify a remote system {#55-verify}

It is recommended to verify each host separately. Follow the steps in chapter [5.3](#53-verify) but use the following yml file:

~~~
- hosts: all
  vars:
    sap_preconfigure_assert: yes
    sap_preconfigure_assert_ignore_errors: yes
    sap_hana_preconfigure_assert: yes
    sap_hana_preconfigure_assert_ignore_errors: yes
  roles:
    - sap-preconfigure
    - sap-hana-preconfigure
~~~

and use the ansible-playbook command line option -l to specify the name of the remote host to verify, as in:

**`# ansible-playbook sap-hana.yml -l host01`**

Notes:

- The correct indentation (e.g. 2 spaces in front of `roles:`) is essential.

- Beginning with rhel-system-roles-sap-3.2.0-1.el8_4, the role names have changed as indicated in chapter [1](#overview).

## 6. Detailed Description {#Detailed-Description}

This chapter describes the RHEL System Roles for SAP in detail.

The purpose of the three roles *sap-preconfigure*, *sap-netweaver-preconfigure*, and *sap-hana-preconfigure* is described in the following table:

| System Role | Purpose
| ----- | ----- |
| *sap-preconfigure* | Install software and perform all configuration steps which are required for the installation of **SAP NetWeaver** and **SAP HANA**. |
| *sap-netweaver-preconfigure* | Install additional software and perform additional configuration steps which are required for **SAP NetWeaver only**. |
| *sap-hana-preconfigure* | Install additional software and perform additional configuration steps which are required for **SAP HANA only**. |

Note: Beginning with rhel-system-roles-sap-3.2.0-1.el8_4, the role names have changed as indicated in chapter [1](#overview).

### 6.1. System Roles and SAP Notes {#61-System-Roles-and-SAP}

The following table contains the System Role and the corresponding action or SAP Note for the RHEL release of the managed node.

<table>
  <tr>
   <td><strong>System Role</strong>
   </td>
   <td><strong>SAP Note for RHEL 7</strong>
   </td>
   <td><strong>SAP Note for RHEL 8</strong>
   </td>
  </tr>
  <tr>
   <td rowspan="3" ><code>sap-preconfigure</code>
   </td>
   <td>SAP Note <a href="https://launchpad.support.sap.com/#/notes/2002167">2002167</a>
   </td>
   <td>SAP Note <a href="https://launchpad.support.sap.com/#/notes/2772999">2772999</a>
   </td>
  </tr>
  <tr>
   <td>SAP Note <a href="https://launchpad.support.sap.com/#/notes/1391070">1391070</a>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>SAP Note <a href="https://launchpad.support.sap.com/#/notes/0941735">0941735</a> (TMPFS only)
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><code>sap-netweaver-preconfigure</code>
   </td>
   <td>SAP Note <a href="https://launchpad.support.sap.com/#/notes/2526952">2526952</a> (tuned profiles only)
   </td>
   <td>SAP Note <a href="https://launchpad.support.sap.com/#/notes/2526952">2526952</a> (tuned profiles only)
   </td>
  </tr>
  <tr>
   <td rowspan="6" ><code>sap-hana-preconfigure</code>
   </td>
   <td>Install required packages as per documents <em>SAP HANA 2.0 running on RHEL 7.x</em> and <em>SAP HANA SPS 12 running on RHEL 7.x</em> which are attached to SAP Note <a href="https://launchpad.support.sap.com/#/notes/2009879">2009879</a>
   </td>
   <td>Install required packages for SAP HANA as mentioned in SAP Note <a href="https://launchpad.support.sap.com/#/notes/2772999">2772999</a>
   </td>
  </tr>
  <tr>
   <td>ppc64le only: Install additional required packages as per <a href="https://www14.software.ibm.com/support/customercare/sas/f/lopdiags/home.html">https://www14.software.ibm.com/support/customercare/sas/f/lopdiags/home.html</a>
   </td>
   <td>ppc64le only: Install additional required packages as per <a href="https://www14.software.ibm.com/support/customercare/sas/f/lopdiags/home.html">https://www14.software.ibm.com/support/customercare/sas/f/lopdiags/home.html</a>
   </td>
  </tr>
  <tr>
   <td>Perform configuration steps as per documents <em>SAP HANA 2.0 running on RHEL 7.x</em> and <em>SAP HANA SPS 12 running on RHEL 7.x</em> which are attached to SAP Note <a href="https://launchpad.support.sap.com/#/notes/2009879">2009879</a>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>ppc64le only: SAP Note <a href="https://launchpad.support.sap.com/#/notes/2055470">2055470</a>
   </td>
   <td>ppc64le only: SAP Note <a href="https://launchpad.support.sap.com/#/notes/2055470">2055470</a>
   </td>
  </tr>
  <tr>
   <td>SAP Note <a href="https://launchpad.support.sap.com/#/notes/2292690">2292690</a>
   </td>
   <td>SAP Note <a href="https://launchpad.support.sap.com/#/notes/2777782">2777782</a>
   </td>
  </tr>
  <tr>
   <td>SAP Note <a href="https://launchpad.support.sap.com/#/notes/2382421">2382421</a>
   </td>
   <td>SAP Note <a href="https://launchpad.support.sap.com/#/notes/2382421">2382421</a>
   </td>
  </tr>
</table>


### 6.2. Implemented SAP Notes {#62-Implemented-SAP-Notes}

The following table contains the SAP Note and its purpose and scope. The RHEL column indicates the specific RHEL releases that the SAP Note supports.

<table>
  <tr>
   <td rowspan="2" ><strong>SAP Note</strong>
   </td>
   <td colspan="2" ><strong>RHEL</strong>
   </td>
   <td rowspan="2" ><strong>Title</strong>
   </td>
   <td rowspan="2" ><strong>Purpose and scope</strong>
   </td>
  </tr>
  <tr>
   <td><strong>7</strong>
   </td>
   <td><strong>8</strong>
   </td>
  </tr>
  <tr>
   <td><a href="https://launchpad.support.sap.com/#/notes/2002167">2002167</a>
   </td>
   <td>X
   </td>
   <td>
   </td>
   <td><em>Red Hat Enterprise Linux 7.x: Installation and Upgrade</em>
   </td>
   <td>General RHEL 7 installation and configuration steps before installing SAP NetWeaver
   </td>
  </tr>
  <tr>
   <td><a href="https://launchpad.support.sap.com/#/notes/1391070">1391070</a>
   </td>
   <td>X
   </td>
   <td>
   </td>
   <td><em>Linux UUID solutions</em>
   </td>
   <td>Installation and configuration of <code>uuidd</code>
   </td>
  </tr>
  <tr>
   <td><a href="https://launchpad.support.sap.com/#/notes/0941735">0941735</a>
   </td>
   <td>X
   </td>
   <td>
   </td>
   <td><em>SAP memory management system for 64-bit Linux systems</em>
   </td>
   <td>SAP and Linux kernel parameters and TMPFS for SAP NetWeaver
   </td>
  </tr>
  <tr>
   <td><a href="https://launchpad.support.sap.com/#/notes/2772999">2772999</a>
   </td>
   <td>
   </td>
   <td>X
   </td>
   <td><em>Red Hat Enterprise Linux 8.x: Installation and Configuration</em>
   </td>
   <td>General RHEL 8 installation and configuration steps, including uuidd, before installing SAP NetWeaver or SAP HANA
   </td>
  </tr>
  <tr>
   <td><a href="https://launchpad.support.sap.com/#/notes/2526952">2526952</a>
   </td>
   <td>X
   </td>
   <td>X
   </td>
   <td><em>Red Hat Enterprise Linux for SAP Solutions</em>
   </td>
   <td>Description of RHEL for SAP Solutions, including tuned-profiles
   </td>
  </tr>
  <tr>
   <td><a href="https://launchpad.support.sap.com/#/notes/2009879">2009879</a>
   </td>
   <td>X
   </td>
   <td>
   </td>
   <td><em>SAP HANA Guidelines for Red Hat Enterprise Linux (RHEL) Operating System</em>
   </td>
   <td>Kernel and OS settings for SAP HANA on RHEL 6.x and RHEL 7.x
   </td>
  </tr>
  <tr>
   <td><a href="https://launchpad.support.sap.com/#/notes/2055470">2055470</a>
   </td>
   <td>X
   </td>
   <td>X
   </td>
   <td><em>HANA on POWER Planning and Installation Specifics - Central Note</em>
   </td>
   <td>Specific installation and configuration steps for SAP HANA on POWER
   </td>
  </tr>
  <tr>
   <td><a href="https://launchpad.support.sap.com/#/notes/2292690">2292690</a>
   </td>
   <td>X
   </td>
   <td>
   </td>
   <td><em>SAP HANA DB: Recommended OS settings for RHEL 7</em>
   </td>
   <td>Specific package requirements, Kernel and OS settings for SAP HANA on RHEL 7.x
   </td>
  </tr>
  <tr>
   <td><a href="https://launchpad.support.sap.com/#/notes/2777782">2777782</a>
   </td>
   <td>
   </td>
   <td>X
   </td>
   <td><em>SAP HANA DB: Recommended OS Settings for RHEL 8</em>
   </td>
   <td>Specific package requirements, Kernel and OS settings for SAP HANA on RHEL 8.x
   </td>
  </tr>
  <tr>
   <td><a href="https://launchpad.support.sap.com/#/notes/2382421">2382421</a>
   </td>
   <td>X
   </td>
   <td>X
   </td>
   <td><em>Optimizing the Network Configuration on HANA- and OS-Level</em>
   </td>
   <td>Network-related kernel settings for SAP HANA
   </td>
  </tr>
</table>

### 6.3. Role variables {#63-Role-Variables}

In each role, default variable settings can be modified to change the behavior of the role. The `README.md` file of each role, located in directory `/usr/share/ansible/roles/<role>`, describes the purpose of these variables as well as their default settings. The variables are defined and can be changed in each role's file `main.yml` in directory `/usr/share/ansible/roles/<role>/defaults` in an inventory file, in your playbooks, or by using the **`ansible-playbook`** command line parameter **`--extra-vars`** or **`-e`**. See the next section for examples.

Some of the variables are described in more detail below to explain their behavior and dependencies:

#### Kernel related variables in `sap-hana-preconfigure`

Kernel variables can be set either in the kernel command line via *grub*, or using *tuned* profile *sap-hana*. Use the following combinations of these variables in `/usr/share/ansible/roles/sap-hana-preconfigure/defaults/main.yml` for the cases described below:

##### Case 1: Use `tuned` profile `sap-hana` only

In case you would like to use `tuned` profile `sap-hana` only, leave the default settings in place:

`sap_hana_preconfigure_use_tuned: yes`

##### Case 2: Use `tuned` profile `sap-hana` and also modify the kernel command line
In case you would like to use tuned and also modify the kernel command line, just modify variable `sap_hana_preconfigure_modify_grub_cmdline_linux` from not set or **`no`** to **`yes`**. By default, this will trigger command `grub2-mkconfig`.
`sap_hana_preconfigure_modify_grub_cmdline_linux:` **`yes`**
##### Case 3: Modify the kernel command line and do not use `tuned`
In case you would like to modify the kernel command line and not switch to `tuned `profile `sap-hana` (this will lead to all kernel settings to be configured statically), change `sap_hana_preconfigure_use_tuned` from **`yes`** to **`no`**. By default, this will trigger the command **`grub2-mkconfig`**.
`sap_hana_preconfigure_use_tuned:` **`no`**

## 7. Examples {#examples}

As a preparation step for these examples, follow the instructions in section *Quick Start*, chapter [Prepare the control node and ssh access to all managed nodes](#prepare_the_control_node) of this document.


#### 7.1. Prepare two systems for SAP HANA  {#71-prep-two-systems}

You want to configure RHEL 7.6 x86_64 server (managed node) `hana-x-76` and RHEL 8.1 ppc64le (POWER9) PowerVM LPAR (managed node) `hana-p-81` for the installation of SAP HANA. You have already verified in SAP note [2235581](https://launchpad.support.sap.com/#/notes/2235581) that these RHEL releases are supported for the SAP HANA version you want to install, and you have also verified that your hardware vendor has certified the hardware for this SAP HANA version. You need to run the roles `sap-preconfigure` and `sap-hana-preconfigure`.

You would like the role `sap-hana-preconfigure` to enable the required repositories for SAP HANA and also set the RHEL minor release to the currently installed level (7.6 and 8.1) so a `yum update` will not cause the managed nodes to be updated beyond these minor releases. You also want the role to update to the latest software level of that minor RHEL release. Instead of the default behavior of the role, which is to not modify `grub` but only use `tuned` to set kernel and other parameters, you would like to use tuned and also modify the boot command line for SAP HANA. And you want the roles to reboot the managed nodes if necessary.

Use the following steps:

1) Verify that there is no production software running on any of the two managed nodes.

2) In case you would like to preserve the original configuration of any of the managed nodes, perform a backup of the server(s). Typically, these roles are run right after RHEL installation, so a backup should not be necessary.

3) Create an inventory file or modify file `/etc/ansible/hosts` so that it contains the following lines (in this example, the hosts have not been made part of a specific host group):
~~~
hana-x-76
hana-p-81
~~~

4) Use some simple commands to verify that you can log in to all three hosts using ssh without password:
`# `**`ssh hana-x-76 uname -a`**
`# `**`ssh hana-p-81 uname -a`**

5) Create a YAML file named **`sap-hana.yml`** with the following content:
~~~
- hosts: all
- vars:
    sap_preconfigure_reboot_ok: yes
    sap_hana_preconfigure_enable_sap_hana_repos: yes
    sap_hana_preconfigure_set_minor_release: yes
    sap_hana_preconfigure_modify_grub_cmdline_linux: yes
    sap_hana_preconfigure_reboot_ok: yes
  roles:
    - sap-preconfigure
    - sap-hana-preconfigure
~~~
Notes:

- The correct indentation (e.g. 2 spaces in front of `roles:`) is essential.

- Beginning with rhel-system-roles-sap-3.2.0-1.el8_4, the role names have changed as indicated in chapter [1](#overview).

6) Run the following ansible-playbook command to configure the two managed nodes. Note that in this example, `- hosts: all` is used in the playbook and there is no group name for the hosts in file `/etc/ansible/hosts`, so the names of the hosts have to be specified after the `-l` command line parameter:

**`# ansible-playbook -l hana-x-76,hana-p-81 sap-hana.yml`**

#### 7.2. Configure four systems for SAP NetWeaver {#72-config-four-systems}

You want to configure the three RHEL 7.7 x86_64 systems (managed nodes) `sap-test`, `sap-qa`, and `sap-prod` (test, QA, and production) and RHEL 8.2 s390x system (managed node) `sap-test-z` for the installation of SAP NetWeaver. For system `sap-test-z`, you do not have access to the root user but to a user with id 0 and with name `root2`. You need to run the roles `sap-preconfigure` and `sap-netweaver-preconfigure`.

The default behavior of the role `sap-preconfigure` is to not update the managed node to the latest RHEL software level but you would llike to update all four managed nodes to the latest RHEL software version. You do not want to fail the roles in case less than 20480 MB of swap space is configured, and you do not want to fail the roles in case a reboot is required. Use the following steps:

1) Verify that there is no other production software running on any of the four managed nodes.

2) In case you would like to preserve the original configuration of any of the servers, perform a backup of the managed nodes(s). Typically, these roles are run right after RHEL installation, so a backup should not be necessary.

3) Create an inventory file or modify file `/etc/ansible/hosts` so that it contains the following lines:
~~~
[sap_netweaver]
sap-test
sap-qa
sap-prod
sap-test-z ansible_user=root2
~~~

4) Use some simple commands to verify that you can log in to all four managed nodes using ssh without password:
`# `**`ssh sap-test uname -a`**
`# `**`ssh sap-qa uname -a`**
`# `**`ssh sap-prod uname -a`**
`# `**`ssh root2@sap-test-z uname -a`**

5) Create a YAML file named **`sap-netweaver.yml`** with the following content:
~~~
- hosts: sap_netweaver
  roles:
    - sap-preconfigure
    - sap-netweaver-preconfigure
~~~
Notes:

- The correct indentation (e.g. 2 spaces in front of `roles:`) is essential.

- Beginning with rhel-system-roles-sap-3.2.0-1.el8_4, the role names have changed as indicated in chapter [1](#overview).

6) Run the following ansible-playbook command to configure the four managed nodes as described above:

`# `**`ansible-playbook sap-netweaver.yml -e "{'sap_preconfigure_update': yes,
'sap_preconfigure_fail_if_reboot_required': no,
'sap_netweaver_preconfigure_fail_if_not_enough_swap_space_configured': no}"`**

7) Reboot all four managed nodes to make sure all required configuration changes are in effect.

#### 7.3. Report the SAP HANA notes compliance status of a RHEL system {#73-report}
You have configured your RHEL 8.2 system `hana-x-82` for SAP HANA manually and you would like to check if your system is compliant with the applicable SAP notes. You need to run the roles `sap-preconfigure` and `sap-hana-preconfigure` in assert mode.
Use the following steps:

1) Create an inventory file or modify file `/etc/ansible/hosts` so that it contains the following lines (in this example, the hosts have not been made part of a specific host group):
`hana-x-82`

2) Use a simple command to verify that you can log in to all the system using ssh without password:

~~~
# ssh hana-x-82 uname -a
~~~
3) Create a YAML file named `sap-hana-assert.yml` with the following content:

~~~
- hosts: all
  vars:
    sap_preconfigure_assert: yes
    sap_preconfigure_assert_ignore_errors: yes
    sap_hana_preconfigure_assert: yes
    sap_hana_preconfigure_assert_ignore_errors: yes
  roles:
    - sap-preconfigure
    - sap-hana-preconfigure
~~~
Notes:

- The correct indentation (e.g. 2 spaces in front of `roles:`) is essential.

- Beginning with rhel-system-roles-sap-3.2.0-1.el8_4, the role names have changed as indicated in chapter [1](#overview).

4) Verify that you are using version 3 or later of the RHEL System Roles for SAP:

- Make sure that you have installed version 3 of the RHEL System Roles for SAP:

~~~
# dnf list installed rhel-system-roles-sap
~~~

- Run the following command:

~~~
# ansible-playbook sap-hana-assert.yml -l hana-x-82 --step -vv
~~~

Answer the first question with "N":
`Perform task: TASK: Gathering Facts (N)o/(y)es/(c)ontinue:` **`N`**
and the second question with "y":
`Perform task: TASK: sap-preconfigure : include os specific vars (N)o/(y)es/(c)ontinue:` **`y`**
This will display the absolute path name of file `tasks/main.yml` and then abort the play (because the vars file could not be found). The path name must be `/usr/share/ansible/roles/sap-preconfigure/tasks/main.yml.`

Sample output:

~~~
TASK [sap-preconfigure : include os specific vars] ****************************************************************************
task path: /usr/share/ansible/roles/sap-preconfigure/tasks/main.yml:3
~~~

5) Run the verification and filter the default ansible-playbook output to only display essential verification information. If you are using a terminal with dark background, replace all occurrences of color code **`[30m `** in the following command sequence by **`[37m.`** Otherwise, the output of some lines will be unreadable (dark font on dark background).

~~~
# ansible-playbook sap-hana-assert.yml | awk '{sub ("    \"msg\": ", "")}
  /TASK/{task_line=$0}
  /fatal:/{fatal_line=$0; nfatal[host]++}
  /...ignoring/{nfatal[host]--; if (nfatal[host]<0) nfatal[host]=0}
  /^[a-z]/&&/: \[/{gsub ("\\[", ""); gsub ("]", ""); gsub (":", ""); host=$2}
  /SAP note/{print "\033[30m[" host"] "$0}
  /FAIL:/{nfail[host]++; print "\033[31m[" host"] "$0}
  /WARN:/{nwarn[host]++; print "\033[33m[" host"] "$0}
  /PASS:/{npass[host]++; print "\033[32m[" host"] "$0}
  /INFO:/{print "\033[34m[" host"] "$0}
  /changed/&&/unreachable/{print "\033[30m[" host"] "$0}
  END{print ("---"); for (var in npass) {printf ("[%s] ", var); if (nfatal[var]>0) {
        printf ("\033[31mFATAL ERROR!!! Playbook might have been aborted!!!\033[30m Last TASK and fatal output:\n"); print task_line, fatal_line
     }
     else printf ("\033[31mFAIL: %d  \033[33mWARN: %d  \033[32mPASS: %d\033[30m\n", nfail[var], nwarn[var], npass[var])}}'
~~~

Sample output:
<pre>
<font color="#CC0000">[hana-x-82] &quot;FAIL: Environment group &apos;server-product-environment&apos; is not installed!&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;uuidd&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;libnsl&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;tcsh&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;psmisc&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;nfs-utils&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;bind-utils&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;compat-sap-c++-9&apos; is installed.&quot;</font>
<font color="#3465A4">[hana-x-82] &quot;INFO: No minimum required package version defined (variable __sap_preconfigure_min_pkgs).&quot;</font>
<font color="#3465A4">[hana-x-82] &quot;INFO: Not checking for possible package updates (variable sap_preconfigure_update).&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: System needs no restart.&quot;</font>
<font color="#2E3436">[hana-x-82] &quot;SAP note 2772999 Step 2: Configure SELinux&quot;</font>
<font color="#3465A4">[hana-x-82] &quot;INFO: When running in normal mode, the role will set the SELinux state to &apos;disabled&apos; (variable sap_preconfigure_selinux_state).&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The system is configured for the SELinux state of &apos;disabled&apos;&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: SELinux is currently disabled.&quot;</font>
[...]
<font color="#2E3436">[hana-x-82] &quot;SAP note 2772999 Step 6: Configure uuidd&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Service &apos;uuidd&apos; is available.&quot;</font>
<font color="#3465A4">[hana-x-82] &quot;INFO: The &apos;uuidd&apos; service is in status &apos;indirect&apos; and in state &apos;stopped&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Service &apos;uuidd.socket&apos; is enabled.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Service &apos;uuidd.socket&apos; is active.&quot;</font>
<font color="#2E3436">[hana-x-82] &quot;SAP note 2772999 Step 7: Configure tmpfs; memtotal_mb = 64161; swaptotal_mb = 25595; sap_preconfigure_size_of_tmpfs_gb = 66&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: An entry for &apos;tmpfs&apos; in /etc/fstab exists.&quot;</font>
<font color="#CC0000">[hana-x-82] &quot;FAIL: The size of tmpfs in /etc/fstab is '7G' but the expected size is '66G!&quot;</font>
<font color="#CC0000">[hana-x-82] &quot;FAIL: The current size of tmpfs is '7G' but the expected size is '66G!&quot;</font>
<font color="#2E3436">[hana-x-82] &quot;SAP note 2772999 Step 8: Configure Linux Kernel Parameters&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: File /etc/sysctl.d/sap.conf exist.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: File /etc/sysctl.d/sap.conf is a regular file.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The value of &apos;vm.max_map_count&apos; in &apos;/etc/sysctl.d/sap.conf&apos; is &apos;2147483647&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The current value of &apos;vm.max_map_count&apos; as per sysctl is &apos;2147483647&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The value of &apos;kernel.pid_max&apos; in &apos;/etc/sysctl.d/sap.conf&apos; is &apos;4194304&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The current value of &apos;kernel.pid_max&apos; as per sysctl is &apos;4194304&apos;.&quot;</font>
<font color="#2E3436">[hana-x-82] &quot;SAP note 2772999 Step 9: Configure Process Resource Limits&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: File /etc/security/limits.d/99-sap.conf exist.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: File /etc/security/limits.d/99-sap.conf is a regular file.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The hard limit of nofile for group &apos;sapsys&apos; in /etc/security/limits.d/99-sap.conf is &apos;65536&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The soft limit of nofile for group &apos;sapsys&apos; in /etc/security/limits.d/99-sap.conf is &apos;65536&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The hard limit of nproc for group &apos;sapsys&apos; in /etc/security/limits.d/99-sap.conf is &apos;unlimited&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The soft limit of nproc for group &apos;sapsys&apos; in /etc/security/limits.d/99-sap.conf is &apos;unlimited&apos;.&quot;</font>
<font color="#2E3436">[hana-x-82] &quot;SAP note 2772999 Step 10: Configure systemd-tmpfiles&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: File /etc/tmpfiles.d/sap.conf exist.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The RHEL release 8.2 is supported for SAP HANA.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Repository &apos;rhel-8-for-x86_64-baseos-e4s-rpms&apos; is enabled.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Repository &apos;rhel-8-for-x86_64-appstream-e4s-rpms&apos; is enabled.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Repository &apos;rhel-8-for-x86_64-sap-solutions-e4s-rpms&apos; is enabled.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The RHEL release is correctly locked to &apos;8.2&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;expect&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;graphviz&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;iptraf-ng&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;krb5-workstation&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;libatomic&apos; is installed.&quot;</font>
[...]
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;xfsprogs&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;gtk2&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;libtool-ltdl&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;tuned-profiles-sap-hana&apos; is installed.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Package &apos;kernel&apos; is already installed as kernel-4.18.0-193.40.1.el8_2 or later. Currently installed latest version: kernel-4.18.0-193.51.1.el8_2.&quot;</font>
<font color="#3465A4">[hana-x-82] &quot;INFO: Not checking for possible package updates (variable sap_hana_preconfigure_update).&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: System needs no restart.&quot;</font>
<font color="#2E3436">[hana-x-82] &quot;SAP note 2777782 Step 2: Configure tuned to use profile sap-hana&quot;</font>
<font color="#3465A4">[hana-x-82] &quot;INFO: The installed version of package tuned is: 2.13.0&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Service &apos;tuned&apos; is available.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Service &apos;tuned&apos; is enabled.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: Service &apos;tuned&apos; is active.&quot;</font>
<font color="#3465A4">[hana-x-82] &quot;INFO: The installed version of package &apos;tuned-profiles-sap-hana&apos; is: 2.13.0&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The tuned profile &apos;sap-hana&apos; is currently active.&quot;</font>
[...]
<font color="#2E3436">[hana-x-82] &quot;SAP note 2777782 Step 10: Increase kernel.pid_max&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: File /etc/sysctl.d/sap.conf exists.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: File /etc/sysctl.d/sap.conf is a regular file.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The value of &apos;kernel.pid_max&apos; in /etc/sysctl.d/sap.conf is &apos;4194304&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The current value of &apos;kernel.pid_max&apos; as per sysctl is &apos;4194304&apos;.&quot;</font>
<font color="#2E3436">[hana-x-82] &quot;SAP note 2777782 Step 11: Enable TSX (Intel Transactional Synchronization Extensions)&quot;</font>
<font color="#2E3436">[hana-x-82] &quot;SAP note 2382421: Recommended network settings for SAP HANA&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: File /etc/sysctl.d/sap_hana.conf exists.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: File /etc/sysctl.d/sap_hana.conf is a regular file.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The value of &apos;net.core.somaxconn&apos; in &apos;/etc/sysctl.d/sap_hana.conf&apos; is &apos;4096&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The current value of &apos;net.core.somaxconn&apos; as per sysctl is &apos;4096&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The value of &apos;net.ipv4.tcp_max_syn_backlog&apos; in &apos;/etc/sysctl.d/sap_hana.conf&apos; is &apos;8192&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The current value of &apos;net.ipv4.tcp_max_syn_backlog&apos; as per sysctl is &apos;8192&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The value of &apos;net.ipv4.tcp_timestamps&apos; in &apos;/etc/sysctl.d/sap_hana.conf&apos; is &apos;1&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The current value of &apos;net.ipv4.tcp_timestamps&apos; as per sysctl is &apos;1&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The value of &apos;net.ipv4.tcp_slow_start_after_idle&apos; in &apos;/etc/sysctl.d/sap_hana.conf&apos; is &apos;0&apos;.&quot;</font>
<font color="#4E9A06">[hana-x-82] &quot;PASS: The current value of &apos;net.ipv4.tcp_slow_start_after_idle&apos; as per sysctl is &apos;0&apos;.&quot;</font>
<font color="#2E3436">[hana-x-82] &quot;SAP note 3024346: Linux Kernel Settings for NetApp NFS&quot;</font>
<font color="#2E3436">[hana-x-82] hana-x-82                   : ok=234  changed=0    unreachable=0    failed=0    skipped=90   rescued=0    ignored=2   </font>
<font color="#2E3436">---</font>
<font color="#2E3436">[hana-x-82] </font><font color="#CC0000">FAIL: 3  </font><font color="#C4A000">WARN: 0  </font><font color="#4E9A06">PASS: 107</font>
</pre>

### 7.4 Example of Setting up a 2 Node HANA Scale-Up cluster

Starting with June 2022 a new set of system roles is available. This allows to setup a 2 node cluster with a single playbook. All necessary parameters are also part of this playbook. Running in an rhev environment you are able to setup an complete cluster using this playbook. The only requirement is the availability of the 


## 8. Related Information {#Related-Information}
- [Linux System Roles upstream project](https://linux-system-roles.github.io/)
- [Red Hat Enterprise Linux (RHEL) System Roles - Red Hat KB Article](https://access.redhat.com/articles/3050101)
- [RHEL System Roles for SAP v.1](https://access.redhat.com/sites/default/files/attachments/rhel_system_roles_for_sapv1.pdf)
- [RHEL System Roles for SAP v.2](https://access.redhat.com/sites/default/files/attachments/rhel_system_roles_for_sapv2_0.pdf)
