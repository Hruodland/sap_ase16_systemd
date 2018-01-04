# sap_ase16_systemd


## Purpose:
Few scripts for a systemd service type unit for  SAS (Sybase) ASE 16.

Use it as a template to clone/fork/copy modify your own.


### Files

|   File              |      Description           |
|----------------|-------|
|sybase.service |Service unit file.|
|sybase_start| Shell script contains the main logic for starting and stopping.|
|README	| This file|



### TODO


Like to have:
- Small Test script.



## Instructions

**Design Notes**:

1. Designed to be a single unit of one dataserver and one backupserver.
1. Servers start asynchronously (this is default behaviour of the  startserver script).
1. Systemd status sybase.service uses showserver.
1. Backupserver is named SYB_BACKUP (otherwise change the code a bit).
1. Shutdown assumes al connections are gone (nowait not used).
1. Uses the User directive in the Unit file so no su or sudo needed in the script.
1. *Developed on Centos7/ SAS ASE  16 (Express edition) for setting up a test and train VM*.


### Installation:

1. Clone  this repo.

1. Adjust scripts to your needs,  see also [Configuration](#Configuration) 
1. Copy the unit script to /etc/systemd/system/  (With permissions: -rw-r--r--.).
1. Copy the sybase_start script to /opt/sybase  path is referred in the unit file).
1. Make script executable  for login sybase.
1. sudo systemctl daemon-reload

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
  1. SERVERPW   Contains the password for database server ADMINUSER.

Examples of changes:

  1. Change it to sap_ase.service.
  1. Add dependencies to other services.
  1. Use a different account to start services (this example also owns the installation).
  1. Get this password from an encrypted password service or script. 
  1. Change the path to the start stop script.
  1. Change the location of the pid file setup in the unit file and script.

---

#### Sample output
```bash
systemctl status sybase.service
```

```no-highlight
● sybase.service - SYBASE ASE
   Loaded: loaded (/etc/systemd/system/sybase.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2018-01-03 13:36:45 CET; 9min ago
  Process: 4547 ExecStop=/opt/sybase/sybase_start stop (code=exited, status=0/SUCCESS)
  Process: 4587 ExecStart=/opt/sybase/sybase_start start (code=exited, status=0/SUCCESS)
 Main PID: 4594 (RUN_FLSRV)
   CGroup: /system.slice/sybase.service
           ├─4594 /bin/sh /opt/sybase/ASE-16_0/install/RUN_FLSRV
           ├─4600 /opt/sybase/ASE-16_0/bin/dataserver -d/opt/sybase/data/mast...
           ├─4603 /bin/sh /opt/sybase/ASE-16_0/install/RUN_FLSRV_BS
           ├─4604 /opt/sybase/ASE-16_0/bin/backupserver -e/opt/sybase/ASE-16_...
           └─4633 /opt/sybase/ASE-16_0/bin/jsagent -p 4900 -n flsrv_centos7 -t 32 -...

Jan 03 13:36:45 flsrv_centos7 systemd[1]: Starting SYBASE ASE...
Jan 03 13:36:45 flsrv_centos7 sybase_start[4587]: Starting sybase_start service:  ....
Jan 03 13:36:45 flsrv_centos7 systemd[1]: Started SYBASE ASE.
```



