# homework23

1. Устнавливаю временную зону на web23

```
[root@web23 ~]# cp /usr/share/zoneinfo/Europe/Moscow /etc/localtime
cp: overwrite '/etc/localtime'? y
```

2. Все ок

```
[root@web23 ~]# date
Tue Jun  7 15:51:24 MSK 2022
```

3. Устанавливаю nginx

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

4. Запускаю nginx

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

5. Проверяю порт

```
[root@web23 ~]# ss -tulpan | grep 80
tcp   LISTEN 0      128           0.0.0.0:80        0.0.0.0:*     users:(("nginx",pid=26195,fd=8),("nginx",pid=26194,fd=8),("nginx",pid=26193,fd=8))
tcp   LISTEN 0      128              [::]:80           [::]:*     users:(("nginx",pid=26195,fd=9),("nginx",pid=26194,fd=9),("nginx",pid=26193,fd=9))
```

6. Открываю в файрволе

```
[root@web23 ~]# firewall-cmd --add-service=https --permanent
success
[root@web23 ~]# firewall-cmd --add-service=http --permanent
success
[root@web23 ~]# firewall-cmd --reload
success
```

7. Проверяю rsyslog на log23

```
[root@log23 ~]# dnf list rsyslog
Last metadata expiration check: 0:55:12 ago on Wed 08 Jun 2022 09:25:57 AM UTC.
Installed Packages
rsyslog.x86_64                                       8.2102.0-7.el8                                           @AppStream
Available Packages
rsyslog.x86_64                                       8.2102.0-7.el8_6.1                                       appstream
```

8. Редактирую rsyslog.conf, включаю прием по tcp и udp, настриваю шаблон файлов 

```
cat /etc/rsyslog.conf
...
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

9. Перезапускаю rsyslog и проверяю, что порты открыты

```
[root@log23 ~]# ss -tulpn | grep 514
udp   UNCONN 0      0            0.0.0.0:514       0.0.0.0:*    users:(("rsyslogd",pid=5425,fd=4))
udp   UNCONN 0      0               [::]:514          [::]:*    users:(("rsyslogd",pid=5425,fd=5))
tcp   LISTEN 0      25           0.0.0.0:514       0.0.0.0:*    users:(("rsyslogd",pid=5425,fd=6))
tcp   LISTEN 0      25              [::]:514          [::]:*    users:(("rsyslogd",pid=5425,fd=7))
```

10. Настриваю файрвол

```
[root@log23 ~]# firewall-cmd --add-port=514/udp --permanent
success
[root@log23 ~]# firewall-cmd --add-port=514/tcp --permanent
success
[root@log23 ~]# firewall-cmd --reload
```

11. В параметра nginx указываю, куда отсылать логи

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

12. Перезапускаю nginx и несколько раз дергаю nginx 

```
[root@web23 ~]# curl 192.168.56.10
```

13. Провеоряю на сервере log23 - логи от nginx летят

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

14. Настраиваю аудит файла /etc/nginx/nginx.conf. Проверяю версию audit

```
[root@web23 ~]# rpm -qa | grep audit
audit-3.0.7-2.el8.2.x86_64
audit-libs-3.0.7-2.el8.2.x86_64
```

15. Прописываю правило аудита

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

16. Перезапускаю аудит и проверяю

```
[root@web23 ~]# service auditd restart
Stopping logging:
Redirecting start to /bin/systemctl start auditd.service
```

17. Проверяю наличе событий в /var/log/audit/audit.log

```
[root@web23 ~]# cat /var/log/audit/audit.log | grep nginx
...

node=web23 type=PATH msg=audit(1654783334.077:1214): item=0 name="/etc/nginx/default.d" inode=134440337 dev=fd:00 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 nametype=NORMAL cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
node=web23 type=SYSCALL msg=audit(1654783371.958:1254): arch=c000003e syscall=82 success=yes exit=0 a0=5595e6a35c00 a1=5595e6a8a520 a2=fffffffffffffe98 a3=81a4 items=4 ppid=7130 pid=7132 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=5 comm="vim" exe="/usr/bin/vim" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
node=web23 type=PATH msg=audit(1654783371.958:1254): item=0 name="/etc/nginx/" inode=850504 dev=fd:00 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 nametype=PARENT cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
node=web23 type=PATH msg=audit(1654783371.958:1254): item=1 name="/etc/nginx/" inode=850504 dev=fd:00 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 nametype=PARENT cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
node=web23 type=PATH msg=audit(1654783371.958:1254): item=2 name="/etc/nginx/nginx.conf" inode=134289980 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 nametype=DELETE cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
node=web23 type=PATH msg=audit(1654783371.958:1254): item=3 name="/etc/nginx/nginx.conf~" inode=134289980 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 nametype=CREATE cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
node=web23 type=SYSCALL msg=audit(1654783371.958:1255): arch=c000003e syscall=257 success=yes exit=5 a0=ffffff9c a1=5595e6a35c00 a2=41 a3=1a4 items=2 ppid=7130 pid=7132 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=5 comm="vim" exe="/usr/bin/vim" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
node=web23 type=PATH msg=audit(1654783371.958:1255): item=0 name="/etc/nginx/" inode=850504 dev=fd:00 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 nametype=PARENT cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
node=web23 type=PATH msg=audit(1654783371.958:1255): item=1 name="/etc/nginx/nginx.conf" inode=607669 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:httpd_config_t:s0 nametype=CREATE cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
node=web23 type=SYSCALL msg=audit(1654783371.992:1256): arch=c000003e syscall=188 success=yes exit=0 a0=5595e6a35c00 a1=7f021f5d9e5e a2=5595e6a88e40 a3=24 items=1 ppid=7130 pid=7132 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=5 comm="vim" exe="/usr/bin/vim" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
node=web23 type=PATH msg=audit(1654783371.992:1256): item=0 name="/etc/nginx/nginx.conf" inode=607669 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:httpd_config_t:s0 nametype=NORMAL cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
node=web23 type=SYSCALL msg=audit(1654783371.992:1257): arch=c000003e syscall=91 success=yes exit=0 a0=5 a1=81a4 a2=7ffe224b4070 a3=24 items=1 ppid=7130 pid=7132 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=5 comm="vim" exe="/usr/bin/vim" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
node=web23 type=SYSCALL msg=audit(1654783371.992:1258): arch=c000003e syscall=188 success=yes exit=0 a0=5595e6a35c00 a1=7f021f18a22f a2=5595e6c765a0 a3=1c items=1 ppid=7130 pid=7132 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=5 comm="vim" exe="/usr/bin/vim" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
node=web23 type=PATH msg=audit(1654783371.992:1258): item=0 name="/etc/nginx/nginx.conf" inode=607669 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 nametype=NORMAL cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
```

18. Настраиваю отправку журналов audit на удаленный сервер. Устанавливаю plugin

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

...
Installed:
  audispd-plugins-3.0.7-2.el8.2.x86_64

Complete!
```

19. Включаю plugin

```
[root@web23 ~]# cat /etc/audit/plugins.d/au-remote.conf

# This file controls the audispd data path to the
# remote event logger. This plugin will send events to
# a remote machine (Central Logger).

active = yes
direction = out
path = /sbin/audisp-remote
type = always
#args =
format = string

```

20. Настраиваю auditd.conf

```
[root@web23 ~]# cat /etc/audit/auditd.conf
..
log_format = RAW
...
name_format = HOSTNAME
....
```

21. Указываю параметры сервер для отправки событий audit

```
[root@web23 ~]# cat /etc/audit/audisp-remote.conf
...
remote_server = 192.168.56.15
port = 60
...

```

22. Открываю порт в firewall


```
[root@log23 ~]# firewall-cmd --add-port=60/tcp --permanent
success
[root@log23 ~]# firewall-cmd --reload
success
```

23. Проверяю наличие событий аудита на log23. События с web23 есть!

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
