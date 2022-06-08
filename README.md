# homework23

```
[root@web23 ~]# cp /usr/share/zoneinfo/Europe/Moscow /etc/localtime
cp: overwrite '/etc/localtime'? y
```


```
[root@web23 ~]# date
Tue Jun  7 15:51:24 MSK 2022
```

```
[root@web23 ~]# dnf install -y nginx
Rocky Linux 8 - AppStream                                                               5.7 MB/s | 8.3 MB     00:01
Rocky Linux 8 - BaseOS                                                                  2.4 MB/s | 2.6 MB     00:01
Rocky Linux 8 - Extras                                                                   30 kB/s |  11 kB     00:00
Extra Packages for Enterprise Linux 8 - x86_64                                          5.1 MB/s |  11 MB     00:02
Extra Packages for Enterprise Linux Modular 8 - x86_64                                  926 kB/s | 1.0 MB     00:01
Dependencies resolved.
========================================================================================================================
 Package                            Architecture  Version                                        Repository        Size
========================================================================================================================
Installing:
 nginx                              x86_64        1:1.14.1-9.module+el8.4.0+542+81547229         appstream        566 k
...
Complete!
```

```
[root@web23 ~]# systemctl start nginx
[root@web23 ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2022-06-07 15:58:16 MSK; 2s ago
  Process: 26191 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 26189 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 26188 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 26193 (nginx)
    Tasks: 3 (limit: 4913)
   Memory: 7.9M
   CGroup: /system.slice/nginx.service
           ├─26193 nginx: master process /usr/sbin/nginx
           ├─26194 nginx: worker process
           └─26195 nginx: worker process

Jun 07 15:58:16 web23 systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jun 07 15:58:16 web23 nginx[26189]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jun 07 15:58:16 web23 nginx[26189]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jun 07 15:58:16 web23 systemd[1]: Started The nginx HTTP and reverse proxy server.
```

```
[root@web23 ~]# ss -tulpan | grep 80
tcp   LISTEN 0      128           0.0.0.0:80        0.0.0.0:*     users:(("nginx",pid=26195,fd=8),("nginx",pid=26194,fd=8),("nginx",pid=26193,fd=8))
tcp   LISTEN 0      128              [::]:80           [::]:*     users:(("nginx",pid=26195,fd=9),("nginx",pid=26194,fd=9),("nginx",pid=26193,fd=9))
```

```
[root@web23 ~]# firewall-cmd --add-service=https --permanent
success
[root@web23 ~]# firewall-cmd --add-service=http --permanent
success
[root@web23 ~]# firewall-cmd --reload
success
```

```
[root@log23 ~]# dnf list rsyslog
Last metadata expiration check: 0:55:12 ago on Wed 08 Jun 2022 09:25:57 AM UTC.
Installed Packages
rsyslog.x86_64                                       8.2102.0-7.el8                                           @AppStream
Available Packages
rsyslog.x86_64                                       8.2102.0-7.el8_6.1                                       appstream
```

```
# Provides UDP syslog reception
# for parameters see http://www.rsyslog.com/doc/imudp.html
module(load="imudp") # needs to be done just once
input(type="imudp" port="514")

# Provides TCP syslog reception
# for parameters see http://www.rsyslog.com/doc/imtcp.html
module(load="imtcp") # needs to be done just once
input(type="imtcp" port="514")
...
#Add remote logs
$template RemoteLogs,"/var/log/rsyslog/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?RemoteLogs
& ~
```
```
[root@log23 ~]# ss -tulpn | grep 514
udp   UNCONN 0      0            0.0.0.0:514       0.0.0.0:*    users:(("rsyslogd",pid=5425,fd=4))
udp   UNCONN 0      0               [::]:514          [::]:*    users:(("rsyslogd",pid=5425,fd=5))
tcp   LISTEN 0      25           0.0.0.0:514       0.0.0.0:*    users:(("rsyslogd",pid=5425,fd=6))
tcp   LISTEN 0      25              [::]:514          [::]:*    users:(("rsyslogd",pid=5425,fd=7))
```
```
[root@log23 ~]# firewall-cmd --add-port=514/udp --permanent
success
[root@log23 ~]# firewall-cmd --add-port=514/tcp --permanent
success
[root@log23 ~]# firewall-cmd --reload
```

```
[root@web23 ~]# cat /etc/nginx/nginx.conf
...
error_log /var/log/nginx/error.log;
error_log syslog:server=192.168.56.15:514,tag=nginx_error;
...

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
    access_log syslog:server=192.168.56.15:514,tag=nginx_access,severity=info combined;


...
```
```
[root@web23 ~]# curl 192.168.56.10
```

```
[root@log23 ~]# cat /var/log/rsyslog/web23/nginx_access.log
Jun  8 15:32:57 web23 nginx_access: 192.168.56.10 - - [08/Jun/2022:15:32:57 +0300] "GET / HTTP/1.1" 200 3429 "-" "curl/7.61.1"
Jun  8 15:32:59 web23 nginx_access: 192.168.56.10 - - [08/Jun/2022:15:32:59 +0300] "GET / HTTP/1.1" 200 3429 "-" "curl/7.61.1"
Jun  8 15:32:59 web23 nginx_access: 192.168.56.10 - - [08/Jun/2022:15:32:59 +0300] "GET / HTTP/1.1" 200 3429 "-" "curl/7.61.1"
Jun  8 15:32:59 web23 nginx_access: 192.168.56.10 - - [08/Jun/2022:15:32:59 +0300] "GET / HTTP/1.1" 200 3429 "-" "curl/7.61.1"
Jun  8 15:33:00 web23 nginx_access: 192.168.56.10 - - [08/Jun/2022:15:33:00 +0300] "GET / HTTP/1.1" 200 3429 "-" "curl/7.61.1"
Jun  8 15:33:00 web23 nginx_access: 192.168.56.10 - - [08/Jun/2022:15:33:00 +0300] "GET / HTTP/1.1" 200 3429 "-" "curl/7.61.1"
[root@log23 ~]#

```

```
[root@web23 ~]# rpm -qa | grep audit
audit-3.0.7-2.el8.2.x86_64
audit-libs-3.0.7-2.el8.2.x86_64
```

```
[root@web23 ~]# cat /etc/audit/rules.d/audit.rules
## First rule - delete all
-D

## Increase the buffers to survive stress events.
## Make this bigger for busy systems
-b 8192

## This determine how long to wait in burst of events
--backlog_wait_time 60000

## Set failure mode to syslog
-f 1
-w /etc/nginx/nginx.conf -p wa -k nginx_conf
-w /etc/nginx/default.d/ -p wa -k nginx_conf
```

```
[root@web23 ~]# service auditd restart
Stopping logging:
Redirecting start to /bin/systemctl start auditd.service
```

```
[root@web23 ~]# dnf install audispd-plugins
Last metadata expiration check: 3:33:18 ago on Wed 08 Jun 2022 12:08:50 PM MSK.
Dependencies resolved.
============================================================================================================================================================================================================================================
 Package                                                       Architecture                                         Version                                                      Repository                                            Size
============================================================================================================================================================================================================================================
Installing:
 audispd-plugins                                               x86_64                                               3.0.7-2.el8.2                                                baseos                                                47 k

Transaction Summary
============================================================================================================================================================================================================================================
Install  1 Package

Total download size: 47 k
Installed size: 72 k
Is this ok [y/N]: y
Downloading Packages:
audispd-plugins-3.0.7-2.el8.2.x86_64.rpm                                                                                                                                                                    176 kB/s |  47 kB     00:00
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                       175 kB/s |  47 kB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                                                    1/1
  Installing       : audispd-plugins-3.0.7-2.el8.2.x86_64                                                                                                                                                                               1/1
  Running scriptlet: audispd-plugins-3.0.7-2.el8.2.x86_64                                                                                                                                                                               1/1
  Verifying        : audispd-plugins-3.0.7-2.el8.2.x86_64                                                                                                                                                                               1/1

Installed:
  audispd-plugins-3.0.7-2.el8.2.x86_64

Complete!
[root@web23 ~]#
```


```
[root@web23 ~]# cat /etc/audit/auditd.conf
#
# This file controls the configuration of the audit daemon
#

local_events = yes
write_logs = yes
log_file = /var/log/audit/audit.log
log_group = root
log_format = RAW
flush = INCREMENTAL_ASYNC
freq = 50
max_log_file = 8
num_logs = 5
priority_boost = 4
name_format = HOSTNAME
##name = mydomain
max_log_file_action = ROTATE
space_left = 75
space_left_action = SYSLOG
verify_email = yes
action_mail_acct = root
admin_space_left = 50
admin_space_left_action = SUSPEND
disk_full_action = SUSPEND
disk_error_action = SUSPEND
use_libwrap = yes
##tcp_listen_port = 60
tcp_listen_queue = 5
tcp_max_per_addr = 1
##tcp_client_ports = 1024-65535
tcp_client_max_idle = 0
transport = TCP
krb5_principal = auditd
##krb5_key_file = /etc/audit/audit.key
distribute_network = no
q_depth = 1200
overflow_action = SYSLOG
max_restarts = 10
plugin_dir = /etc/audit/plugins.d
end_of_event_timeout = 2
```

```
[root@web23 ~]# cat /etc/audit/audisp-remote.conf
#
# This file controls the configuration of the audit remote
# logging subsystem, audisp-remote.
#

remote_server = 192.168.56.15
port = 60
##local_port =
transport = tcp
queue_file = /var/spool/audit/remote.log
mode = immediate
queue_depth = 10240
format = managed
network_retry_time = 1
max_tries_per_record = 3
max_time_per_record = 5
heartbeat_timeout = 0

network_failure_action = stop
disk_low_action = ignore
disk_full_action = warn_once
disk_error_action = warn_once
remote_ending_action = reconnect
generic_error_action = syslog
generic_warning_action = syslog
queue_error_action = stop
overflow_action = syslog
startup_failure_action = warn_once_continue

##krb5_principal =
##krb5_client_name = auditd
##krb5_key_file = /etc/audisp/audisp-remote.key

```

```
[root@log23 ~]# firewall-cmd --add-port=60/tcp --permanent
success
[root@log23 ~]# firewall-cmd --reload
success
```

```
[root@log23 ~]# cat /var/log/audit/audit.log | grep web
...
node=web23 type=PROCTITLE msg=audit(1654693294.461:1263): proctitle=76696D002F6574632F6E67696E782F6E67696E782E636F6E66
node=web23 type=SYSCALL msg=audit(1654693294.489:1264): arch=c000003e syscall=188 success=yes exit=0 a0=5596b7ed2b90 a1=7f7ac5f58e5e a2=5596b8115970 a3=24 items=1 ppid=27585 pid=28173 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=6 comm="vim" exe="/usr/bin/vim" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
node=web23 type=CWD msg=audit(1654693294.489:1264): cwd="/root"
node=web23 type=PATH msg=audit(1654693294.489:1264): item=0 name="/etc/nginx/nginx.conf" inode=784835 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:httpd_config_t:s0 nametype=NORMAL cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
node=web23 type=PROCTITLE msg=audit(1654693294.489:1264): proctitle=76696D002F6574632F6E67696E782F6E67696E782E636F6E66
node=web23 type=SYSCALL msg=audit(1654693294.489:1265): arch=c000003e syscall=91 success=yes exit=0 a0=5 a1=81a4 a2=7ffd250e4560 a3=24 items=1 ppid=27585 pid=28173 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=6 comm="vim" exe="/usr/bin/vim" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
node=web23 type=CWD msg=audit(1654693294.489:1265): cwd="/root"
node=web23 type=PATH msg=audit(1654693294.489:1265): item=0 name=(null) inode=784835 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 nametype=NORMAL cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
node=web23 type=PROCTITLE msg=audit(1654693294.489:1265): proctitle=76696D002F6574632F6E67696E782F6E67696E782E636F6E66
node=web23 type=SYSCALL msg=audit(1654693294.489:1266): arch=c000003e syscall=188 success=yes exit=0 a0=5596b7ed2b90 a1=7f7ac5b0922f a2=5596b8126680 a3=1c items=1 ppid=27585 pid=28173 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=6 comm="vim" exe="/usr/bin/vim" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
node=web23 type=CWD msg=audit(1654693294.489:1266): cwd="/root"
node=web23 type=PATH msg=audit(1654693294.489:1266): item=0 name="/etc/nginx/nginx.conf" inode=784835 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 nametype=NORMAL cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
node=web23 type=PROCTITLE msg=audit(1654693294.489:1266): proctitle=76696D002F6574632F6E67696E782F6E67696E782E636F6E66
[root@log23 ~]#
```
