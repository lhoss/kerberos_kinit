kerberos_kinit
==============

This ansible role, in short provides a scripted kinit with keytabs including support to dynamically lookup the keytab (based on a glob pattern).

This functionality is tailored to get a kerberos ticket for service accounts in a Cloudera cluster.
In particular the role default variables contain predefined directory glob patterns to lookup the keytab of cloudera-agent managed services (like HDFS).


Requirements
------------

This role does not depend on any extra modules or other roles.


Role Variables
--------------

All the role vars explained in short, including their defaults. (However the code is the best doc!):

```
# The kerberos realm
kerberos_kinit_realm: EXAMPLE.LOCAL

# The username under which kinit is run
kerberos_kinit_user: "{{ ansible_user }}"

# The kerberos principal. By default set from the username and the host's fqdn
kerberos_kinit_principal: "{{ kerberos_kinit_user }}/{{ ansible_fqdn }}"

# Optional kinit options. By default empty
kerberos_kinit_opts:

# The full path to the keytab OR a glob pattern, which will be scanned (using "ls -1t $glob") and the first result used as the keytab.
kerberos_kinit_keytab_path: ...

# Flag to enable the dynamic keytab lookup. By default it is enabled IF the keytab_path contains a '*'
kerberos_kinit_dynamic_keytab: "{{ kerberos_kinit_keytab_path is search('\\*') }}"

# Selects one of the supported keytab config types. Only relevant if you do not override the `kerberos_kinit_keytab_path` directly
kerberos_kinit_keytab_type: hdp

```


Examples
----------------

An example playbook, to run kinit for the hdfs service user, on a Cloudera CDH Cluster:
```
- hosts: "{{ hosts | default('all') }}"
  gather_facts: yes
  become: yes
  run_once: true
  tasks:
    - import_role:
        name: kerberos_kinit
      vars:
        #kerberos_kinit_realm: "EXAMPLE.LOCAL"
        kerberos_kinit_user: "hdfs"
        kerberos_kinit_keytab_type: cdh 
      when: do_kinit | default(True) | bool
      tags: kinit
    
    - name: deploying hdfs home-folders
      block:
        ...
```
