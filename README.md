[![Ansible](https://img.shields.io/badge/Ansible-%3E%3D2.4-blue.svg)](https://www.ansible.com/)
[![MongoDB(Percona)](https://img.shields.io/badge/MongoDB(Percona)-v3.4-orange.svg)](https://www.percona.com/software/mongo-database/percona-server-for-mongodb)
[![MongoDB(Origin)](https://img.shields.io/badge/MongoDB(Origin)-v3.4-green.svg)](https://www.mongodb.com/)
[![MongoDB(Origin)](https://img.shields.io/badge/MongoDB(Origin)-v3.6-green.svg)](https://www.mongodb.com/)

# Ansible MongoDB role with origin and percona installation version's
This ansible role for MongoDB contains origin and percona installation version's.

Most difference between origin and percona version is:
 - percona version has in-memory storage engine wich is wired-tiger engine running in RAM without disk usage (origin version have this engine only in enterprise commercial version)
 - percona version has audit log which can help to view what any user doing in db (origin version have this engine only in enterprise commercial version)
 
### Features
 - allows to run more then one independent MongoDB instance with individual config settings
 - uses dynamic RAM calculation to limit consumed resources
 - create defined db's
 - create defined admin's and other user's
 - slack notification on installation failed
 - each MongoDB instance service running under **numactl** for best perfomance
 - install script for disabling huge pages and add it to autostart

### Requirements
 - ubuntu >= 14.04 (Debian version not tested yet)
 - ansible version >= 2.4
 - install packages:
     - python-jmespath (```sudo apt get install -y python-jmespath```)
     - python-psycopg2 (```sudo apt get install -y python-psycopg2```)
 - before run any playbooks replace **hash_behavior** variable in the /etc/ansible/ansible.cfg from
  replace to merge.
 - it may be necessary to remove previous MongoDB installation (especially if ansible role is crashes on installation step).
On Debian/Ubuntu run this commands:
   - **percona version**
     - ```sudo apt-get autoremove -y percona-server-mongodb*```
     - ```sudo apt-get purge -y percona-server-mongodb*```
   - **origin version**
     - ```sudo apt-get autoremove -y mongodb-org*```
     - ```sudo apt-get purge -y mongodb-org*```
     
### Configuring
Inventory file must contain this variables (you can define your variable values):
```
#=========================== Local variables =============================
default_app_path: "/opt/app"
default_data_path: "/opt/data"
#=========================================================================
default:
  path:
    config: "/etc"
    data: "{{ default_data_path }}"
    log: "/opt/logs"
    pid: "/var/run"
    
slack:
  enabled: "no" # no | yes
  token: "<slack-token>"
  channel: "<slack-channel>"
  username: "Ansible Agent"
  link_names: 0
  emoji:
    icon: ":secret:"
  attachment:
    info:
      line:
        color: "#2d9ee0"
    success: 
      line:
        color: "good"
    warning:
      line:
        color: "warning"
    error:
      line:
        color: "danger"
```
Also we can override in invertory any variable from the **defaults** vars file.
For example:
 - add second in-memory MongoDB instance in invertory file:
 ```
#=========================== Local variables =============================
mongodb_ram_pre_calculated: "{{ (os_ram_size_gb | int) * 0.08 }}"
mongodb_ram_default_ram_size_gb: "{{ (mongodb_ram_pre_calculated if (mongodb_ram_pre_calculated | int) > 0 else 1) | int }}"
mongodb_ram_name: "mongodb-ram"
#=========================================================================

mongodb:
  install:
    maintainer: "percona" # percona | origin
    # Current role contains version 3.4 and 3.6 for origin version
    # and only 3.4 for percona version
    version: "3.4"
  # Instances must has a key and instance configuration
  # i.e main | ram | cache | other
  instances:
    # For example change main instance port
    main:
      net:
        port: 28017
    # Second MongoDB instance with in-memory percona storage.
    # WARNING! Data erase on system reboot.
    ram:
      name: "{{ mongodb_ram_name }}" # Mongodb instance in RAM for temporaly data
      service:
        name: "{{ mongodb_ram_name }}.service"
        autostart:
          enabled: true
      config:
        name: "{{ mongodb_ram_name }}.conf"
        path: "{{ default.path.config }}/{{ mongodb_main_name }}"
      net:
        ip: "127.0.0.1"
        port: 27018
        http:
          enabled: false     
        ipv6: false             
        maxconns: 65536   
      process:
        management:
          fork: true             
      storage:
        dbpath: "{{ default.path.data }}/{{ mongodb_ram_name }}"
        engine: 
          name: "inMemory" # wiredTiger | mmapv1 | inMemory (only percona version)
        wired_tiger:
          config:
            cache:
              size: "{{ mongodb_ram_default_ram_size_gb }}" # Gb
        # In-memory engine available in enterprise and percona
        # version only of the MongoDB.
        in_memory:
          config:
            in_memory:
              size: "{{ mongodb_ram_default_ram_size_gb }}" # Gb
            statistics:
              log_delay_secs: 0
        mmapv1:
          prealloc_data_files: true
          ns_size: 2048 # Each namespace have file size as 2Gb
          quota:
            enforced: false
            max_files_per_db: 16 # Number of quota files per DB
          small_files: false # Very useful for non-data nodes
        journal:
          enabled: false 
      auth:
        enabled: true 
      profiling:
        mode: "off" # off | slowOp | all
        slowop:
          threshold: 10000 # In millis
      log:
        destination: "file"
        append: true
        name: "{{ mongodb_ram_name }}.log"
        path: "{{ default.path.log }}/{{ mongodb_main_name }}"
        # Audit log available only in percona and enterprise commercial version
        audit:
          destination: "file"
          name: "{{ mongodb_ram_name }}.audit.json.log"
          path: "{{ default.path.log }}/{{ mongodb_main_name }}"
          format: "JSON" # JSON | BSON (Bson most perfomance choice but required special tools for viewing)
          filter: "" # Filter string not added to config if empty
      init_db_list:
        - "test"
      init_user_list:
        admins:
          - db: "admin"
            name: "admin"
            password: "password"
            roles: "userAdminAnyDatabase"
        users:
          - db: "test"
            name: "user"
            password: "password"
            roles: "readWrite,dbAdmin"
      # All values must be quoted also true|false
      parameters_list: {
        auditAuthorizationSuccess: "true" # This parameter only for percona version of MongoDB
      }
 ```
     
### Running
**After successfull installing system reboot required!!!**

Go to the playbook directory and run:
```
sudo ansible-playbook -i <inventory-host-file> <playbook-file>.yml
```
By default all config file's located in **/etc/mongodb**, all logs in **/opt/logs/mongodb**, db's data located in **/opt/data/mongodb** and **/opt/data/mongodb-ram** (if second instance named as ram is using)

We can run our two instances (service name defined in mongodb.instances.*.name variable):
  - ```sudo service mongodb (start|stop|restart)```
  - ```sudo service mongodb-ram (start|stop|restart)```

### Tasks tags
Mongo role contain tags:
```
  tags:
    - "install-packages" # Can be used when full installation with other roles required
    - "install-mongo" # Can be used when only MongoDB installation required
    - "update-configs" # Can be used when only config with other roles required
    - "update-config-mongo" # Can be used when only MongoDB config update required
```
