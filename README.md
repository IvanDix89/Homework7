# Homework7

1.Ставим необходимые пакеты и качаем исходники.

        yum install -y \redhat-lsb-core \wget \rpmdevtools \rpm-build \createrepo \yum-utils
        wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm
        wget https://www.openssl.org/source/latest.tar.gz
        
        
2.Распаковываем

        tar -xvf latest.tar.gz
        
3.Ставим зависимости.

        yum-builddep rpmbuild/SPECS/nginx.spec
        
4.Редактируем nginx.spec

        --with-openssl=/root/openssl-1.1.1g
        
5.Собираем RPM пакет и проверяем

        rpmbuild -bb rpmbuild/SPECS/nginx.spec
        
        Выполняется(%clean): /bin/sh -e /var/tmp/rpm-tmp.rxP8pi
        + umask 022
        + cd /root/rpmbuild/BUILD
        + cd nginx-1.14.1
        + /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/nginx-1.14.1-1.el7_4.ngx.x86_64
        + exit 0


6.Проверяем пакеты

        ll rpmbuild/RPMS/x86_64/
        
        -rw-r--r--. 1 root root 2155128 апр 22 15:34 nginx-1.14.1-1.el7_4.ngx.x86_64.rpm
        -rw-r--r--. 1 root root 2528156 апр 22 15:34 nginx-debuginfo-1.14.1-1.el7_4.ngx.x86_64.rpm
        
 7.Устанавливаем пакет и проверяем
 
        yum localinstall -y \ rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm

        systemctl start nginx
        systemctl status nginx
        ● nginx.service - nginx - high performance web server
           Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
           Active: active (running) since Ср 2020-04-22 15:40:11 UTC; 11s ago
           
8.Приступаем к созданию репозитория, создадим каталог скопируем туда наш пакет и пакет Percona-Server

        mkdir /usr/share/nginx/html/repo
        cp rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm /usr/share/nginx/html/repo/
        wget http://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noarch.rpm -O /usr/share/nginx/html/repo/percona-release-0.1-6.noarch.rpm

9.Инициализируем репозиторий.

        createrepo /usr/share/nginx/html/repo/
        Spawning worker 0 with 2 pkgs
        Workers Finished
        Saving Primary metadata
        Saving file lists metadata
        Saving other metadata
        Generating sqlite DBs
        Sqlite DBs complete
        
10.Редактируем default.conf и перезакускаем nginx

        nginx -t
        nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
        nginx: configuration file /etc/nginx/nginx.conf test is successful
        nginx -s reload

11.Проверяем curl -a http://localhost/repo/

        <html>
        <head><title>Index of /repo/</title></head>
        <body bgcolor="white">
        <h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
        <a href="repodata/">repodata/</a>                                          22-Apr-2020 15:44                   -
        <a href="nginx-1.14.1-1.el7_4.ngx.x86_64.rpm">nginx-1.14.1-1.el7_4.ngx.x86_64.rpm</a>                22-Apr-2020 15:43             2155128
        <a href="percona-release-0.1-6.noarch.rpm">percona-release-0.1-6.noarch.rpm</a>                   13-Jun-2018 06:34               14520
        </pre><hr></body>
        </html>
        
 11.Добавляем репозиторий
 
         cat >> /etc/yum.repos.d/otus.repo << EOF
         [otus]
         name=otus-linux
         baseurl=http://localhost/repo
         gpgcheck=0
         enabled=1
         EOF
 
 12.Убедимся что репозиторий подключен и с ним все ок.
 
         [root@centos yum.repos.d]# yum repolist enabled | grep otus
        otus                                otus-linux                                 2

         [root@centos yum.repos.d]# yum list | grep otus
        nginx.x86_64                                1:1.14.1-1.el7_4.ngx       otus     
        percona-release.noarch                      0.1-6                      otus 
 




