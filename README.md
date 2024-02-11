# Otus_15_PAM

## Цель домашнего задания
Научиться создавать пользователей и добавлять им ограничения

## Описание домашнего задания

Запретить всем пользователям, кроме группы admin, логин в выходные (суббота и воскресенье), без учета праздников


Создаём пользователя otusadm и otus:

```
maksimzv@maksimzvnb:~/Otus_15$ vagrant ssh
[vagrant@pam ~]$ sudo -i
[root@pam ~]# sudo useradd otusadm && sudo useradd otus
```

Создаём пользователям пароли:

```
[root@pam ~]# echo "Otus2022" | sudo passwd --stdin otusadm && echo "Otus2022" | sudo passwd --stdin otus
Changing password for user otusadm.
passwd: all authentication tokens updated successfully.
Changing password for user otus.
passwd: all authentication tokens updated successfully.
```

Создаём группу admin и добавляем пользователей vagrant,root и otusadm в группу admin:

```
[root@pam ~]# cat /etc/group | grep admin
printadmin:x:997:
admin:x:1003:otusadm,root,vagrant
[root@pam ~]# vi /usr/local/bin/login.sh
```

После создания пользователей, нужно проверить, что они могут подключаться по SSH к нашей ВМ. Для этого пытаемся подключиться с хостовой машины:

```
aksimzv@maksimzvnb:~$ ssh otus@192.168.57.10
The authenticity of host '192.168.57.10 (192.168.57.10)' can't be established.
ED25519 key fingerprint is SHA256:Wxsf59V7z8oXseS6LJ/pg9TgS/KP8tcxcAHRkPw9gPQ.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? y
Please type 'yes', 'no' or the fingerprint: yes
Warning: Permanently added '192.168.57.10' (ED25519) to the list of known hosts.
otus@192.168.57.10's password: 
[otus@pam ~]$ whoami
otus
[otus@pam ~]$ exit
logout
Connection to 192.168.57.10 closed.
maksimzv@maksimzvnb:~$ ssh otusadm@192.168.57.10
otusadm@192.168.57.10's password: 
[otusadm@pam ~]$ whoami
otusadm
[otusadm@pam ~]$ exit
logout
Connection to 192.168.57.10 closed.
```

Проверим, что пользователи root, vagrant и otusadm есть в группе admin:

```
[root@pam ~]# cat /etc/group | grep admin
printadmin:x:997:
admin:x:1003:otusadm,root,vagrant
```
Создадим файл-скрипт /usr/local/bin/login.sh

```
[root@pam ~]# nano /usr/local/bin/login.sh
#!/bin/bash
if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun"]; then
 if getent group admin | grep -qw "$PAM_USER"; then
        exit 0
      else
	exit 1
    fi
  else
    exit 0
fi
[root@pam ~]# chmod +x /usr/local/bin/login.sh
```

Укажем в файле /etc/pam.d/sshd модуль pam_exec и наш скрипт:

```
auth       required     pam_exec.so /usr/local/bin/login.sh
```

Так как делаю ДЗ в выходной день, то проверить возможно:

```
aksimzv@maksimzvnb:~$ ssh otus@192.168.57.10
otus@192.168.57.10's password: 
/usr/local/bin/login.sh failed: exit code 1
Connection closed by 192.168.57.10 port 22
```
