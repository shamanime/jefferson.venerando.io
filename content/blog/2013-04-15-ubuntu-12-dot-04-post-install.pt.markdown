---
categories:
- Ubuntu
- nginx
comments: true
date: 2013-04-15T00:00:00Z
title: Ubuntu 12.04 post install
url: /2013/04/15/ubuntu-12-dot-04-post-install/
---

## Configuração inicial
Logue como root:

{{< highlight sh >}}
ssh root@10.0.0.1
{{< / highlight >}}

Configure o hostname:

{{< highlight sh >}}
# echo "plato" > /etc/hostname
# hostname -F /etc/hostname
{{< / highlight >}}

Altere o arquivo `/etc/hosts`:

{{< highlight sh >}}
127.0.0.1                       localhost.localdomain    localhost
12.34.56.78                     plato.example.com        plato
2600:3c01::a123:b456:c789:d012  plato.example.com        plato
{{< / highlight >}}

Configure a timezone para `America/Sao_Paulo`:

{{< highlight sh >}}
# dpkg-reconfigure tzdata
{{< / highlight >}}

Atualize o software:

{{< highlight sh >}}
# apt-get update
# apt-get upgrade --show-upgraded
{{< / highlight >}}

Instale alguns pacotes:

{{< highlight sh >}}
# apt-get -y install linux-headers-$(uname -r) build-essential
# apt-get -y install zlib1g-dev libssl-dev libreadline-gplv2-dev
# apt-get -y install imagemagick
# apt-get -y install libmagickwand-dev
# apt-get -y install libcurl4-openssl-dev
# apt-get -y install vim
{{< / highlight >}}

## Usuáiro padrão
Crie um usuário padrão:

{{< highlight sh >}}
# adduser shaman
{{< / highlight >}}

Adicione o usuário no grupo de sudo:

{{< highlight sh >}}
# usermod -aG sudo shaman
{{< / highlight >}}

Configure o SSH no arquivo `/etc/ssh/sshd_config`:

{{< highlight sh >}}
PermitRootLogin no
{{< / highlight >}}

Reinicie o serviço SSH:

{{< highlight sh >}}
# service ssh restart
{{< / highlight >}}

Localmente gere seu par de chaves se ainda não tiver:

{{< highlight sh >}}
ssh-keygen
{{< / highlight >}}

Copie as chaves para o host:

{{< highlight sh >}}
ssh-copy-id -i ~/.ssh/id_rsa.pub shaman@10.0.0.1
{{< / highlight >}}

Entre na máquina sem root:

{{< highlight sh >}}
ssh shaman@10.0.0.1
{{< / highlight >}}

## RVM, MySQL e NodeJS
Instale RVM system wide:

{{< highlight sh >}}
$ \curl -#L https://get.rvm.io | sudo bash -s stable --autolibs=3
{{< / highlight >}}

Adicione o usuário no grupo `rvm` (depois saia e conecte novamente):

{{< highlight sh >}}
$ sudo usermod -aG rvm shaman
{{< / highlight >}}

Crie o arquivo `/etc/gemrc`:

{{< highlight sh >}}
install: --no-rdoc --no-ri
update:  --no-rdoc --no-ri
{{< / highlight >}}

Instale o Ruby 1.9.3-p392:

{{< highlight sh >}}
$ rvm install ruby-1.9.3-p392
{{< / highlight >}}

Instale o MySQL:

{{< highlight sh >}}
$ sudo apt-get -y install mysql-server mysql-client mysql-common libmysqlclient-dev
{{< / highlight >}}

Instale o NodeJS:

{{< highlight sh >}}
$ sudo apt-get -y install software-properties-common
$ sudo apt-get -y install python-software-properties python g++ make
$ sudo add-apt-repository ppa:chris-lea/node.js
$ sudo apt-get update
$ sudo apt-get -y install nodejs
{{< / highlight >}}

## Passenger e Nginx
Instale o passenger:

{{< highlight sh >}}
$ gem install passenger
{{< / highlight >}}

Baixe e extraia os módulos pra upload:

(http://wiki.nginx.org/NginxHttpUploadProgressModule)

{{< highlight sh >}}
$ wget -O upload-progress https://github.com/masterzen/nginx-upload-progress-module/tarball/v0.9.0
$ tar -zxf v0.9.0
$ mv masterzen-nginx-upload-progress-module-a788dea/ progress-module
{{< / highlight >}}

Baixe e extraia o Nginx:

{{< highlight sh >}}
$ wget http://nginx.org/download/nginx-1.3.15.tar.gz
$ tar -zxf nginx-1.3.15.tar.gz
{{< / highlight >}}

Crie um usuário para o Nginx:

{{< highlight sh >}}
$ sudo adduser --system --no-create-home --disabled-login --disabled-password \
--group nginx
{{< / highlight >}}

Instale o Nginx com o Passenger:

{{< highlight sh >}}
$ rvmsudo passenger-install-nginx-module
{{< / highlight >}}

Escolha a opção `2`, e passe os seguintes argumentos na instalação:

{{< highlight sh >}}
--user=nginx --group=nginx --add-module=/home/shaman/tools/progress-module
{{< / highlight >}}

Crie um script de inicialização para o Nginx:

{{< highlight sh >}}
$ wget -O init-deb.sh http://library.linode.com/assets/660-init-deb.sh
$ sudo mv init-deb.sh /etc/init.d/nginx
$ sudo chmod +x /etc/init.d/nginx
$ sudo /usr/sbin/update-rc.d -f nginx defaults
$ sudo /etc/init.d/nginx start
{{< / highlight >}}

Edite o arquivo `/opt/nginx/nginx.conf` e coloque dentro do bloco `http {}`:

{{< highlight sh >}}
include sites-enabled/*;
{{< / highlight >}}

Crie as pastas necessárias:

{{< highlight sh >}}
$ sudo mkdir /opt/nginx/conf/sites-available
$ sudo mkdir /opt/nginx/conf/sites-enabled
{{< / highlight >}}

Para adicionar um site:

{{< highlight sh >}}
$ cd /opt/nginx/conf/sites-available
$ sudo touch app.com
$ sudo ln -s /opt/nginx/conf/sites-available/app.com /opt/nginx/conf/sites-enabled/
$ sudo /etc/init.d/nginx restart
{{< / highlight >}}
