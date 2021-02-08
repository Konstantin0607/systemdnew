#1.Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова.
# Файл и слово должны задаваться в /etc/sysconfig.

#Создаем файл конфигурации для сервиса

            touch /etc/sysconfig/watchlog

#Зполняем его

           # Configuration file for my watchdog service
           # Place it to /etc/sysconfig
           # File and word in that file that we will be monit
           WORD="ALERT"
           LOG=/var/log/watchlog.log


#Затем создаем /var/log/watchlog.log и пишем туда строку ALERT

            touch /var/log/watchlog.log


#Далее создаем скрипт nano /opt/watchlog.sh

            #!/bin/bash
            WORD=$1
            LOG=$2
            DATE=`date`
            if grep $WORD $LOG &> /dev/null
            then
            logger "$DATE: I found word, Master!" # logger отправляет лог в системный журнал (/var/log/messages)
            else
            exit 0
            fi


           chmod +x /opt/watchlog.sh


#Далее создаем unit для сервиса

           nano /etc/systemd/system/watchlog.service

           [Unit]
           Description=My watchlog service
           [Service]
           Type=oneshot
           EnvironmentFile=/etc/sysconfig/watchlog
           ExecStart=/opt/watchlog.sh $WORD $LOG

#Создаем unit для таймера, timer файл

           nano /etc/systemd/system/watchlog.timer

           [Unit]
           Description=Run watchlog script every 30 second
           [Timer]
           # Run every 30 second
           OnUnitActiveSec=30
           Unit=watchlog.service
           [Install]
           WantedBy=multi-user.target


#Затем достаточно только стартануть timer:

           systemctl enable watchlog.timer

           systemctl start watchlog.timer


#Выполняем команду

           tail -f /var/log/messages


              Feb  8 09:11:55 localhost systemd: Stopping User Slice of vagrant.
              Feb  8 09:12:17 localhost systemd: Created slice User Slice of vagrant.
              Feb  8 09:12:17 localhost systemd: Starting User Slice of vagrant.
              Feb  8 09:12:17 localhost systemd: Started Session 4 of user vagrant.
              Feb  8 09:12:17 localhost systemd-logind: New session 4 of user vagrant.
              Feb  8 09:12:17 localhost systemd: Starting Session 4 of user vagrant.
              Feb  8 09:13:42 localhost yum[3770]: Installed: nano-2.3.1-10.el7.x86_64
              Feb  8 09:22:03 localhost systemd: Reloading.
              Feb  8 09:22:25 localhost systemd: Started Run watchlog script every 30 second.
              Feb  8 09:22:25 localhost systemd: Starting Run watchlog script every 30 second.
              Feb  8 09:23:07 localhost systemd: Starting My watchlog service...
              Feb  8 09:23:07 localhost root: Mon Feb  8 09:23:07 UTC 2021: I found word, Master!
              Feb  8 09:23:07 localhost systemd: Started My watchlog service.



#2.Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл.
# Имя сервиса должно так же называться

#Устанавливаем

           yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y


#etc/rc.d/init.d/spawn-fcg - скрипт, который будет переписан

#Заходим в /etc/sysconfig/spawn-fcgi и приводим его к следующему виду:

           nano /etc/sysconfig/spawn-fcgi


           # You must set some working options before the "spawn-fcgi" service will work.
           # If SOCKET points to a file, then this file is cleaned up by the init script.
           #
           # See spawn-fcgi(1) for all possible options.
           #
           # Example :
           SOCKET=/var/run/php-fcgi.sock
           OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"


#Создаем unit файл

           nano /etc/systemd/system/spawn-fcgi.service

    
           [Unit]
           Description=Spawn-fcgi startup service by Otus
           After=network.target
           [Service]
           Type=simple
           PIDFile=/var/run/spawn-fcgi.pid
           EnvironmentFile=/etc/sysconfig/spawn-fcgi
           ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
           KillMode=process
           [Install]
           WantedBy=multi-user.target

#проверяем что все успешно работает,выполняем команды

           systemctl start spawn-fcgi

           systemctl status spawn-fcgi


            spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2021-02-08 09:31:53 UTC; 9s ago
 Main PID: 4107 (php-cgi)
   CGroup: /system.slice/spawn-fcgi.service
           +-4107 /usr/bin/php-cgi
           +-4108 /usr/bin/php-cgi
           +-4109 /usr/bin/php-cgi
           +-4110 /usr/bin/php-cgi
           +-4111 /usr/bin/php-cgi
           +-4112 /usr/bin/php-cgi
           +-4113 /usr/bin/php-cgi
           +-4114 /usr/bin/php-cgi
           +-4115 /usr/bin/php-cgi
           +-4116 /usr/bin/php-cgi
           +-4117 /usr/bin/php-cgi
           +-4118 /usr/bin/php-cgi
           +-4119 /usr/bin/php-cgi
           +-4120 /usr/bin/php-cgi
           +-4121 /usr/bin/php-cgi
           +-4122 /usr/bin/php-cgi
           +-4123 /usr/bin/php-cgi
           +-4124 /usr/bin/php-cgi
           +-4125 /usr/bin/php-cgi
           +-4126 /usr/bin/php-cgi
           +-4127 /usr/bin/php-cgi
           +-4128 /usr/bin/php-cgi
           +-4129 /usr/bin/php-cgi
           +-4130 /usr/bin/php-cgi
           +-4131 /usr/bin/php-cgi
           +-4132 /usr/bin/php-cgi
           +-4133 /usr/bin/php-cgi
           +-4134 /usr/bin/php-cgi
           +-4135 /usr/bin/php-cgi
           +-4136 /usr/bin/php-cgi
           +-4137 /usr/bin/php-cgi
           +-4138 /usr/bin/php-cgi
           L-4139 /usr/bin/php-cgi

Feb 08 09:31:53 otuslinux systemd[1]: Started Spawn-fcgi startup service by Otus.
Feb 08 09:31:53 otuslinux systemd[1]: Starting Spawn-fcgi startup service by Otus


#3.Дополнить юнит-файл apache httpd возможностьб запустить несколько инстансов сервера с разными конфигами:

#Копируем файл из /usr/lib/systemd/system/

           cp /usr/lib/systemd/system/httpd.service /etc/systemd/system

#переименновываем его

           mv /etc/systemd/system/httpd.service /etc/systemd/system/httpd@.service


#приводим его к данному виду

           [Unit]
           Description=The Apache HTTP Server
           After=network.target remote-fs.target nss-lookup.target
           Documentation=man:httpd(8)
           Documentation=man:apachectl(8)
           [Service]

           Type=notify
           EnvironmentFile=/etc/sysconfig/httpd-%I #(%I - это и есть подстановка)
           ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
           ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
           ExecStop=/bin/kill -WINCH ${MAINPID}
           KillSignal=SIGCONT
           PrivateTmp=true

           [Install]
           WantedBy=multi-user.target


#Переходим в каталог с конфиругационными файлами и путем копирования делаем 2 конфига
#в нашем случае это будут first.conf и second.conf

           scp httpd.conf first.conf
           scp httpd.conf second.conf



#Для удачного запуска, в конфигурационных файлах должны быть указаны уникальные для каждого экземпляра опции Listen 8080 и путь до PidFile.


           nano second.conf
      
#PidFile /var/run/httpd-second.pid - т.е. должен быть указан файл пида
#Listen 8080 - указан порт, который будет отличаться от другого инстанса



           systemctl status httpd@first.service

           systemctl status httpd@second.service


#Переходим в sysconfig
#Таким же путем создаем два файла


           scp httpd httpd-first
           scp httpd httpd-second



#меняем файл httpd-first 

           #OPTIONS=-f conf/first.conf


#меняем файл httpd-second

           #OPTIONS=-f conf/second.conf


#Запускаем сервис с первым конфигурационным файлом и проверяем статус


                systemctl start httpd@first.service
                systemctl status httpd@first.service



            httpd@first.service - The Apache HTTP Server
   Loaded: loaded (/etc/systemd/system/httpd@.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2021-02-08 09:53:32 UTC; 13s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 4833 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/system-httpd.slice/httpd@first.service
           +-4833 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           +-4834 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           +-4835 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           +-4836 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           +-4837 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           +-4838 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           L-4839 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND



#Запускаем сервис со вторым конфигурационным файлом и проверяем статус


           systemctl start httpd@second.service
           systemctl status httpd@second.service


            httpd@second.service - The Apache HTTP Server
   Loaded: loaded (/etc/systemd/system/httpd@.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2021-02-08 10:02:38 UTC; 12s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 4863 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/system-httpd.slice/httpd@second.service
           +-4863 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           +-4864 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           +-4865 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           +-4866 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           +-4867 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           +-4868 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
           L-4869 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
#Проверяем порты на которых работают сервисы


           ss -tnlp


           State      Recv-Q Send-Q                                       Local Address:Port                                                      Peer Address:Port
           LISTEN     0      128                                                      *:111                                                                  *:*                   users:(("rpcbind",pid=621,fd=8))
           LISTEN     0      128                                                      *:22                                                                   *:*                   users:(("sshd",pid=943,fd=3))
           LISTEN     0      100                                              127.0.0.1:25                                                                   *:*                   users:(("master",pid=1144,fd=13))
           LISTEN     0      128                                                     :::111                                                                 :::*                   users:(("rpcbind",pid=621,fd=11))
           LISTEN     0      128                                                     :::8080                                                                :::*                   users:(("httpd",pid=4869,fd=4),("httpd",pid=4868,fd=4),("httpd",pid=4867,fd=4),("httpd",pid=4866,fd=4),("httpd",pid=4865,fd=4),("httpd",pid=4864,fd=4),("httpd",pid=4863,fd=4))
           LISTEN     0      128                                                     :::80                                                                  :::*                   users:(("httpd",pid=4839,fd=4),("httpd",pid=4838,fd=4),("httpd",pid=4837,fd=4),("httpd",pid=4836,fd=4),("httpd",pid=4835,fd=4),("httpd",pid=4834,fd=4),("httpd",pid=4833,fd=4))
           LISTEN     0      128                                                     :::22                                                                  :::*                   users:(("sshd",pid=943,fd=4))
           LISTEN     0      100                                                    ::1:25                                                                  :::*                   users:(("master",pid=1144,fd=14))











            










            
