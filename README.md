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

