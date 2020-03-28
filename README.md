# Пользователи и группы. Авторизация и аутентификация  (Centos 7)
-----------------------------------------------------------------------

- Создаем группу ```groupadd admin```
- Добавляем туда пользователя ```usermod -aG admin test_user```
- Устанавливаем компонент, с помощью которого мы сможем описывать правила входа в виде обычного bash скрипта ```yum install pam_script -y```
- Можем посмотреть какие файлы использует этот компонент ```rpm -ql pam_script```
```
[root@lvm ~]# rpm -ql pam_script
/etc/pam-script.d
/etc/pam_script
/etc/pam_script_acct
/etc/pam_script_auth
/etc/pam_script_passwd
/etc/pam_script_ses_close
/etc/pam_script_ses_open
/lib64/security/pam_script.so
/usr/share/doc/pam_script-1.1.8
/usr/share/doc/pam_script-1.1.8/AUTHORS
/usr/share/doc/pam_script-1.1.8/COPYING
/usr/share/doc/pam_script-1.1.8/ChangeLog
/usr/share/doc/pam_script-1.1.8/NEWS
/usr/share/doc/pam_script-1.1.8/README
/usr/share/doc/pam_script-1.1.8/README.pam_script
/usr/share/man/man7/pam-script.7.gz
```
- Приводим файл ```/etc/pam.d/sshd``` к следующему виду и добавляем строку ```auth       required     pam_script.so```:
```
#%PAM-1.0

auth       required     pam_script.so # Добавляем сюда эту строку
auth       required     pam_sepermit.so
auth       substack     password-auth
auth       include      postlogin
# Used with polkit to reauthorize users in remote sessions
-auth      optional     pam_reauthorize.so prepare
account required pam_access.so
account required pam_time.so
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      password-auth
session    include      postlogin
# Used with polkit to reauthorize users in remote sessions
-session   optional     pam_reauthorize.so prepare
```
- Далее приводим файл ```/etc/pam_script``` к следующему виду: (у файла должны быть права на исполнение)
```
#!/bin/bash

if [[ `grep $PAM_USER /etc/group | grep 'admin'` ]]
then
exit 0
fi
if [[ `date +%u` > 5 ]]
then
exit 1
fi
```
В этом файле мы проверяем состоит ли пользователь в группе admin и, если да, то пускаем его (значит его можно пускать всегда). Если он не состоит в этой группе то срабатывает проверка на то, какой сейчас день недели, если он больше 5, т.е. выходные, то не пускаем.

Проверяем доступ:

![Image alt](https://github.com/MuTalKing/pam/blob/master/Pam1.png)

Убираем пользователя из группы admin командой ```sudo gpasswd -d test_user admin```

И получаем такой результат:

![Image alt](https://github.com/MuTalKing/pam/blob/master/Pam2.png)
