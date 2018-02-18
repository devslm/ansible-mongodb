# Ansible MongoDB role with origin and percona installation version's
This ansible role for MongoDB contains origin and percona installation version's.

Most difference between origin and percona version is:
 - percona version has in-memory storage engine wich is wired-tiger engine running in RAM without disk usage (origin version have this engine only in enterprise commercial version)
 - percona version has audit log wich can help to view what any user doing in db (origin version have this engine only in enterprise commercial version)
 
### Features
 - allows to run more then one independent MongoDB instance with individual config settings
 - uses dynamic RAM calculation to limit consumed resources
 - create defined db's
 - create defined admin's and other user's
 - slack notification on installation failed

### Requirements
It may be necessary to remove previous MongoDB installation (especially if ansible role is crashes on installation step).
On Debian/Ubuntu run this commands:
 - **percona version**
   - sudo apt-get autoremove -y percona-server-mongodb*
   - sudo apt-get purge -y percona-server-mongodb*
 - **origin version**
   - sudo apt-get autoremove -y mongodb-org*
   - sudo apt-get purge -y mongodb-org*
