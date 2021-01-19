

## Summary

I've been following 
* the latest Linkedin Learning ansible course from there general DevOps path https://www.linkedin.com/learning/learning-ansible-2
* then going more in depth with "Ansible 101 by Jeff Geerling" (both youtube and e-book) https://www.jeffgeerling.com/blog/2020/ansible-101-jeff-geerling-youtube-streaming-series

I'm pleased now to be running with vagrant and ansible provisioning. The initial example (from geerlingguy) was to install NTP via a role:

```
---
- hosts: all
  become: true 
  tasks:
  - name: Ensure NTP (for time synchronization) is installed.
    yum: name=ntp state=present
  - name: Ensure NTP is running.
    service: name=ntpd state=started enabled=yes
```

Once I had this working my next step was to install postgres via a role downloaded from ansible galaxy.


## Deploying postgres via ansible 

Ref:
* nice summary info https://github.com/ANXS/postgresql
* general introduction/tutorial https://severalnines.com/database-blog/postgresql-deployment-and-maintenance-ansible

### Installing Ansible role “anxs.postgresql” on my ansible control node (my ubuntu laptop called ijsselstein)

```
dpitts@ijsselstein:~/projects/vagrant-ansible-for-devops$ ansible-galaxy  install anxs.postgresql
- downloading role 'postgresql', owned by anxs
- downloading role from https://github.com/ANXS/postgresql/archive/v1.11.1.tar.gz
- extracting anxs.postgresql to /home/dpitts/.ansible/roles/anxs.postgresql
- anxs.postgresql (v1.11.1) was installed successfully
```
and then checking the defaults
```
dpitts@ijsselstein:~/projects/vagrant-ansible-for-devop-3nodes$ grep -E '^postgresql_(version|port|max_connections):' ~/.ansible/roles/anxs.postgresql/defaults/main.yml
postgresql_version: 11
postgresql_port: 5432
postgresql_max_connections: 100
```


### Updating my playbook.yml to include new role anxs.postgresql (downloaded from ansible galaxy)

```
dpitts@ijsselstein:~/projects/vagrant-ansible-for-devops$ git diff playbook.yml
diff --git a/playbook.yml b/playbook.yml
index 465be4a..5aec7bc 100644
--- a/playbook.yml
+++ b/playbook.yml
@@ -1,6 +1,8 @@
 ---
 - hosts: all
   become: true 
+  roles: 
+  - role: anxs.postgresql
   tasks:
   - name: Ensure NTP (for time synchronization) is installed.
     yum: name=ntp state=present
```


### Installing postgresql on my ansible managed node (a vagrant/virtualbox vm running centos7)

A few things to highlight here:
* ansible dynamically skipped apt (as the managed node is running centos7) and so chose yum
* there are lots of futher configuration options to be explored/used e.g PostgreSQL extensions, databases and users.
```
dpitts@ijsselstein:~/projects/vagrant-ansible-for-devops$ vagrant provision
==> default: Running provisioner: ansible...
Vagrant has automatically selected the compatibility mode '2.0'
according to the Ansible version installed (2.9.16).

Alternatively, the compatibility mode can be specified in your Vagrantfile:
https://www.vagrantup.com/docs/provisioning/ansible_common.html#compatibility_mode

    default: Running ansible-playbook...

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [default]

TASK [anxs.postgresql : include_vars] ******************************************
ok: [default] => (item=/home/dpitts/.ansible/roles/anxs.postgresql/vars/../vars/RedHat.yml)

TASK [anxs.postgresql : PostgreSQL | Make sure the CA certificates are available | apt] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Add PostgreSQL repository apt-key | apt] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Add PostgreSQL repository | apt] **********
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Add PostgreSQL repository preferences | apt] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Make sure the dependencies are installed | apt] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Install PostgreSQL | apt] *****************
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | PGTune | apt] *****************************
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Install all the required dependencies | yum] ***
changed: [default]

TASK [anxs.postgresql : PostgreSQL | Add PostgreSQL repository | yum] **********
changed: [default]

TASK [anxs.postgresql : PostgreSQL | Make sure the dependencies are installed | yum] ***
ok: [default]

TASK [anxs.postgresql : PostgreSQL | Install PostgreSQL | yum] *****************
changed: [default]

TASK [anxs.postgresql : PostgreSQL | Setup service users profile | yum] ********
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Create ~/pgtab.example | yum] *************
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Create ~/pgtab header | yum] **************
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Create ~/pgtab Ansible warning | yum] *****
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Add database to ~/pgtab | yum] ************
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | PGTune | yum] *****************************
skipping: [default]

TASK [anxs.postgresql : PostgrSQL | Install all the required depedencies | dnf] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Add yum Repository | dnf] *****************
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Install PostgreSQL | dnf] *****************
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | PGTune | dnf] *****************************
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Extensions | Make sure the postgres contrib extensions are installed | Debian] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Extensions | Make sure the postgres contrib extensions are installed | RedHat] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Extensions | Make sure the postgres contrib extensions are installed | Fedora] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Extensions | Make sure the development headers are installed | Debian] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Extensions | Make sure the development headers are installed | RedHat] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Extensions | Make sure the development headers are installed | Fedora] ***
skipping: [default]

TASK [anxs.postgresql : include_vars] ******************************************
skipping: [default] => (item=/home/dpitts/.ansible/roles/anxs.postgresql/vars/../vars/empty.yml) 

TASK [anxs.postgresql : PostgreSQL | Extensions | Make sure the postgis extensions are installed | Debian] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Extensions | Make sure the postgis extensions are installed | RedHat] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Extensions | Make sure the postgis extensions are installed | Fedora] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | FDW | Load OS specific variables] *********
ok: [default]

TASK [anxs.postgresql : PostgreSQL | FDW | MySQL] ******************************
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | FDW | OGR] ********************************
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Drop the data directory | RedHat] *********
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Make sure the postgres data directory exists] ***
ok: [default]

TASK [anxs.postgresql : PostgreSQL | Make sure the postgres WAL directory exists] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Make sure the postgres log directory exists] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Ensure the locale for lc_collate and lc_ctype is generated | Debian] ***
skipping: [default] => (item=en_US.UTF-8) 
skipping: [default] => (item=en_US.UTF-8) 

TASK [anxs.postgresql : PostgreSQL | Ensure the locale is generated | RedHat] ***
ok: [default] => (item={u'parts': [u'en_US', u'UTF-8'], u'locale_name': u'en_US.UTF-8'})
ok: [default] => (item={u'parts': [u'en_US', u'UTF-8'], u'locale_name': u'en_US.UTF-8'})

TASK [anxs.postgresql : PostgreSQL | Stop PostgreSQL | Debian] *****************
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Reset the cluster - drop the existing one | Debian] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Reset the cluster - create a new one (with specified encoding and locale) | Debian] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Update systemd | Debian] ******************
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Start PostgreSQL | Debian] ****************
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Check whether the postgres data directory is initialized | RedHat] ***
ok: [default]

TASK [anxs.postgresql : PostgreSQL | Initialize the database | RedHat] *********
[WARNING]: Module remote_tmp /var/lib/pgsql/.ansible/tmp did not exist and was
created with a mode of 0700, this may cause issues when running as another
user. To avoid this, create the remote_tmp dir with the correct permissions
manually
changed: [default]

TASK [anxs.postgresql : PostgreSQL | Verify postgresql cluster version] ********
ok: [default]

TASK [anxs.postgresql : PostgreSQL | Ensure configuration directory exists] ****
changed: [default]

TASK [anxs.postgresql : PostgreSQL | Update configuration - pt. 1 (pg_hba.conf)] ***
changed: [default]

TASK [anxs.postgresql : PostgreSQL | Update configuration - pt. 2 (postgresql.conf)] ***
changed: [default]

TASK [anxs.postgresql : PostgreSQL | Update configuration - pt. 3 (pgtune)] ****
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Update configuration - pt. 4 (pg_ident.conf)] ***
changed: [default]

TASK [anxs.postgresql : PostgreSQL | Create folder for additional configuration files] ***
changed: [default]

TASK [anxs.postgresql : PostgreSQL | Ensure the systemd directory for PostgreSQL exists | RedHat] ***
changed: [default]

TASK [anxs.postgresql : PostgreSQL | Use the conf directory when starting the Postgres service | RedHat] ***
changed: [default]

TASK [anxs.postgresql : PostgreSQL | Ensure the pid directory for PostgreSQL exists] ***
changed: [default]

TASK [anxs.postgresql : PostgreSQL | Reload all conf files] ********************
changed: [default]

TASK [anxs.postgresql : PostgreSQL | Ensure PostgreSQL is running] *************
ok: [default]

TASK [anxs.postgresql : PostgreSQL | Make sure the PostgreSQL users are present] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Ensure PostgreSQL is running] *************
ok: [default]

TASK [anxs.postgresql : PostgreSQL | Make sure the PostgreSQL databases are present] ***

TASK [anxs.postgresql : PostgreSQL | Add extensions to the databases] **********

TASK [anxs.postgresql : PostgreSQL | Add hstore to the databases with the requirement] ***

TASK [anxs.postgresql : PostgreSQL | Add uuid-ossp to the database with the requirement] ***

TASK [anxs.postgresql : PostgreSQL | Add postgis to the databases with the requirement] ***

TASK [anxs.postgresql : PostgreSQL | add cube to the database with the requirement] ***

TASK [anxs.postgresql : PostgreSQL | Add plpgsql to the database with the requirement] ***

TASK [anxs.postgresql : PostgreSQL | add earthdistance to the database with the requirement] ***

TASK [anxs.postgresql : PostgreSQL | Add citext to the database with the requirement] ***

TASK [anxs.postgresql : PostgreSQL | Add Schema to databases] ******************

TASK [anxs.postgresql : PostgreSQL | Update the user privileges] ***************

TASK [anxs.postgresql : PostgreSQL | (Monit) Copy the postgresql monit service file] ***
skipping: [default]

TASK [anxs.postgresql : PostgreSQL | Check binary version] *********************
ok: [default]

TASK [anxs.postgresql : PostgreSQL | Check database version] *******************
ok: [default]

TASK [anxs.postgresql : PostgreSQL | Verify binary and database versions match] ***
skipping: [default]

TASK [Ensure NTP (for time synchronization) is installed.] *********************
ok: [default]

TASK [Ensure NTP is running.] **************************************************
changed: [default]

RUNNING HANDLER [anxs.postgresql : restart postgresql] *************************
changed: [default]

PLAY RECAP *********************************************************************
default                    : ok=28   changed=15   unreachable=0    failed=0    skipped=53   rescued=0    ignored=0   
```

### finally checking (via systemctl) the postgresql-11.service is running on our managed nodes 
now we can see the `/usr/pgsql-11/bin/postmaster` (pid 4520) running under postgresql-11.service in systemctl

```
dpitts@ijsselstein:~/projects/vagrant-ansible-for-devops$ vagrant ssh
Last login: Tue Jan 19 09:37:05 2021 from 10.0.2.2
[vagrant@localhost ~]$ sudo -i
[root@localhost ~]# systemctl status
● localhost.localdomain
    State: degraded
     Jobs: 0 queued
   Failed: 1 units
    Since: di 2021-01-19 09:35:28 UTC; 2min 31s ago
   CGroup: /
           ├─1 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
           ├─user.slice
           │ └─user-1000.slice
           │   ├─session-3.scope
           │   │ ├─4539 sshd: vagrant [priv]
           │   │ ├─4542 sshd: vagrant@pts/0 
           │   │ ├─4543 -bash
           │   │ ├─4560 sudo -i
           │   │ ├─4562 -bash
           │   │ ├─4575 systemctl status
           │   │ └─4576 systemctl status
           │   └─session-2.scope
           │     ├─2096 sshd: vagrant [priv]
           │     └─2102 sshd: vagrant@notty 
           └─system.slice
             ├─postgresql-11.service
             │ ├─4520 /usr/pgsql-11/bin/postmaster -D /etc/postgresql/11/data
             │ ├─4523 postgres: data: checkpointer                           
             │ ├─4524 postgres: data: background writer                      
             │ ├─4525 postgres: data: walwriter                              
             │ ├─4526 postgres: data: autovacuum launcher                    
             │ ├─4527 postgres: data: stats collector                        
             │ └─4528 postgres: data: logical replication launcher           
             ├─ntpd.service
             │ └─4418 /usr/sbin/ntpd -u ntp:ntp -g
             ├─vboxadd-service.service
             │ └─1035 /usr/sbin/VBoxService --pidfile /var/run/vboxadd-service.sh
             ├─tuned.service
             │ └─981 /usr/bin/python2 -Es /usr/sbin/tuned -l -P
             ├─postfix.service
             │ ├─1206 /usr/libexec/postfix/master -w
             │ ├─1215 pickup -l -t unix -u
             │ └─1216 qmgr -l -t unix -u
             ├─rsyslog.service
             │ └─975 /usr/sbin/rsyslogd -n
             ├─sshd.service
             │ └─974 /usr/sbin/sshd -D
             ├─crond.service
             │ └─685 /usr/sbin/crond -n
             ├─polkit.service
             │ └─659 /usr/lib/polkit-1/polkitd --no-debug
             ├─NetworkManager.service
             │ ├─658 /usr/sbin/NetworkManager --no-daemon
             │ └─710 /sbin/dhclient -d -q -sf /usr/libexec/nm-dhcp-helper -pf /var/run/dhclient-enp0s3.pid -lf /var/lib/NetworkManager/d
             ├─gssproxy.service
             │ └─667 /usr/sbin/gssproxy -D
             ├─systemd-logind.service
             │ └─653 /usr/lib/systemd/systemd-logind
             ├─dbus.service
             │ └─645 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
             ├─rpcbind.service
             │ └─646 /sbin/rpcbind -w
             ├─auditd.service
             │ └─622 /sbin/auditd
             ├─systemd-udevd.service
             │ └─494 /usr/lib/systemd/systemd-udevd
             ├─lvm2-lvmetad.service
             │ └─480 /usr/sbin/lvmetad -f
             ├─system-getty.slice
             │ └─getty@tty1.service
             │   └─693 /sbin/agetty --noclear tty1 linux
             └─systemd-journald.service
               └─464 /usr/lib/systemd/systemd-journald
lines 35-69/69 (END)
```
