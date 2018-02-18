# Ansible MongoDB role
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
