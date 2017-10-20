---
categories:
- Passenger
- Capistrano
- Deploy
comments: true
date: 2013-04-16T00:00:00Z
title: Passenger, Capistrano e environment variables
url: /2013/04/16/passenger-capistrano-e-environment-variables/
---

## Capistrano
Para usar env vars com Capistrano, abra o arquivo `/etc/ssh/sshd_config` e adicione a seguinte linha:

{{< highlight sh >}}
PermitUserEnvironment yes
{{< / highlight >}}

Não se esqueça de reiniciar o serviço:

{{< highlight sh >}}
$ sudo service ssh restart
{{< / highlight >}}

Agora é só coloar suas variáveis no arquivo `~/.ssh/environment` do usuário deployer (sem o export antes)!

## RVM
Se você usa RVM, crie o arquivo `/usr/local/my_ruby_wrapper_script` com o conteúdo:

{{< highlight sh >}}
#!/bin/sh
export MYSQL_WEB_PROD_DB="db"
export MYSQL_WEB_PROD_USR_NAME="usuario"
export MYSQL_WEB_PROD_USR_PASS="senha"
exec "/usr/local/rvm/rubies/ruby-1.9.3-p392/bin/ruby" "$@"
{{< / highlight >}}

Troque o parâmetro do exec para o caminho do seu ruby.

Dê as permissões para o arquivo:

{{< highlight sh >}}
$ sudo chmod +x /usr/local/my_ruby_wrapper_script
{{< / highlight >}}

E troque altere a linha do `nginx.conf` (ou da sua config do Apache) para:

{{< highlight sh >}}
passenger_ruby /usr/local/my_ruby_wrapper_script;
{{< / highlight >}}

Reinicie o Nginx e Passenger, e pronto!
