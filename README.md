# sap_ase16_systemd


## Purpose:
Few scripts for a systemd service type unit and rotate logs  for  SAP (Sybase) ASE 16 database server and replication server on a single linux instance.
This automates starting, stopping and log cleanup.

Use it as an example to clone/fork/copy/split/modify your own.

### License
MIT


### Files and Directories

|   File              |      Description           |
|----------------|-------|
|sybase.service |Service unit file.|
|sybase_start| Shell script contains the main logic for starting and stopping.|
|logrotate.d/sapase16|Rotate logs example for an ASE instance | 
|README	| This file|




|  Dir             |      Description           |
|------------------|----------------------------|
|/opt/sybase/sybase_start_PID.d/ |PID file directory|



### TODO
-

## Instructions

**Design Notes**:

1. Designed to be a single unit of one dataserver and one backupserver and one repserver.
1. Servers start asynchronously (this is default behaviour of the  startserver script).
1. Backupserver is logically named SYB_BACKUP (otherwise change the code a bit).
1. Shutdown assumes al connections are gone (nowait not used).
1. Uses the User directive in the Unit file so no su or sudo needed in the script.
1. Replication server start assumes there is a rep agent running on the RSSD database. 
1. Developed on Centos7/ SAP ASE  16 (Express edition) for setting up a test VM.


### Installation:

1. Clone  this repo.

1. Adjust scripts to your needs, see also [Configuration](#Configuration) 
1. Copy the unit script to /etc/systemd/system/  (With permissions: -rw-r--r--.).
1. Copy the sybase_start script to /opt/sybase path is referred in the unit file).
1. Make script executable  for login sybase.
1. sudo systemctl daemon-reload
1. Optionall: The logrotate file goes into /etc/logrotate.d (Centos/Redhat), verify path to log file!

Do some testing:

1. systemctl start sybase.service (su - sybase first)
1. systemctl status sybase.service 
1. systemctl stop sybase.service  (su - sybase first)

While testing check the errorlog for your database server.

*Finally*:

If ok, reboot Linux and test database connectivity (allow some time to bring databases online).

#### Configuration:

Configuration variables:

  1. SYBASE     Installation path software .
  1. ADMINUSER  Contains name of the database admin login (sa by default).
  1. SERVERPW   Contains the password for database server ADMINUSER. (Concider using openssl and encypted password files or a product like vault for storing secrets).
  1. Script assumes passwords for ADMINUSER are identical on ASE and REP.
  1. SERVER  Name of the SAP ASE server
  1. REPSERVER Name of the Replication server instance (Take the ID server).

Examples of changes:

  1. Change it to sap_ase.service.
  1. Use a different account to start services (this example also owns the installation).
  1. Get this password from an encrypted password service or script. 
  1. Change the path to the start stop script.
  1. Change the location of the pid file setup in the unit file and script.

---

#### Sample output
```bash
systemctl status sybase.service
```


Notice that this will NOT invoke the bash script  with the status argument, this is how systemd behaves. For detailed status info invoke the script *sybase_start status*.

```no-highlight
● sybase.service - SYBASE ASE
   Loaded: loaded (/etc/systemd/system/sybase.service; enabled; vendor preset: disabled)
   Active: activating (start) since Fri 2018-02-09 08:58:11 CET; 3min 50s ago
  Process: 1167 ExecStart=/opt/sybase/sybase_start start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/sybase.service
           ├─1252 /bin/sh /opt/sybase/ASE-16_0/install/RUN_SRV
           ├─1262 /opt/sybase/ASE-16_0/bin/dataserver -d/opt/sybase/data/master.dat -e/opt/sybase/ASE-16_0/install/srv.log -c/opt/sybase/ASE-16_0/SRV.cfg -M/opt/sybase/ASE-16_0 -N/opt/sybase/ASE-16_0/...
           ├─1374 /bin/sh /opt/sybase/ASE-16_0/install/RUN_SRV_BS
           ├─1375 /bin/bash /opt/sybase/sybase_start start
           ├─1378 tail -f /opt/sybase/ASE-16_0/install/SRV.log
           ├─1379 tail -n +1
           ├─1393 /opt/sybase/ASE-16_0/bin/backupserver -e/opt/sybase/ASE-16_0/install/srv_BS.log -N25 -C20 -I/opt/sybase/interfaces -M/opt/sybase/ASE-16_0/bin/sybmultbuf -Ssrv_BS
           ├─1909 /opt/sybase/ASE-16_0/bin/jsagent -p 4900 -n srv -t 32 -l 0 -v -e /opt/sybase/ASE-16_0/install/srv_JSAGENT.log -i /opt/sybase > /dev/null
           ├─1922 sh -c /opt/sybase/REP-15_5/install/RUN_PRS & 
           └─1923 /opt/sybase/REP-15_5/bin/repserver -SPRS -C/opt/sybase/REP-15_5/install/PRS.cfg -E/opt/sybase/REP-15_5/install/PRS.log -I/opt/sybase/interfaces

Feb 09 08:58:11 srv systemd[1]: Starting SYBASE ASE...

Feb 09 08:58:14 srv sybase_start[1167]: Starting srv service:  pid: 1252  Success.
Feb 09 08:58:15 srv sybase_start[1167]: Starting srv backup service:  Success.
Feb 09 08:58:15 srv sybase_start[1167]: Waiting for repagent on RSSD
Feb 09 08:59:38 srv sybase_start[1167]: 00:0006:00000:00015:2018/02/09 08:59:37.98 server  Started Rep Agent on database, 'PRS_RSSD' (dbid = 7)

```



