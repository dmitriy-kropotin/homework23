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
error_log syslog:server=192.168.50.15:514,tag=nginx_error;
#access_log syslog:server=192.168.50.15:514,tag=nginx_access,severity=info combined;
pid /run/nginx.pid;
...

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
    access_log syslog:server=192.168.50.15:514,tag=nginx_access,severity=info combined;


...
```
