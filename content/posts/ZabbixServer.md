---
title: "Tutorial de como instalar e configurar o Zabbix Server e o Grafana - Parte 1"
date: 2024-12-22T00:59:19-03:00
draft: true
categories:
  - Tutorial
author: "Manoel Augusto"
tags:
  - Zabbix
  - Grafana
  - Monitoramento
description: "Aprenda a instalar e configurar o Zabbix Server e o Grafana, no primeiro passo dessa série de tutoriais."
---


# Tutorial de como instalar e configurar o Zabbix Server e o Grafana - Parte 1

E aí, pessoal! Vamos colocar o Zabbix para trabalhar e, de quebra, juntar ele com o Grafana para deixar o monitoramento ainda mais top? Bora lá!

#### Pré-requisitos:

- Servidor Debian ou Ubuntu, na versão mais atual, com acesso à internet e senha de root para acesso durante a instalação.

- Requisitos para a VM ou servidor, segundo a própria documentação do Zabbix, para até 1000 clientes sendo monitorados: idealmente, 2 vCPUs, 8 GB de memória e 50 a 60 GB de armazenamento.

- O servidor deve ter acesso à internet, caso contrário, a instalação não será possível.

### Criando o ambiente

```bash
su -
```

Após inserir a senha do usuário root, lembrando que este tutorial está sendo feito no Debian 12, o próximo passo será configurar o que chamamos de LAMP (**L**inux, **A**pache, **M**ariaDB ou **M**ySQL e **P**HP).

#### 1º Passo: Atualizar a máquina:

```bash
apt update &&  apt upgrade -y
```

#### 2º Passo: Instalaremos de uma única vez o Apache, MariaDB e PHP.

```bash
apt install -y apache2 mariadb-server php libapache2-mod-php php-cli php-fpm php-json php-pdo php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath
```

Você pode verificar se todos foram instalados corretamente com os comandos abaixo, conferindo as saídas:

Verificar PHP:

```bash
php -v
```

![](/home/manoel/bitsdeconhecimento/static/image2.png)

Verificar MariaDB:

```bash
mysql --version
```

![](/home/manoel/bitsdeconhecimento/static/image3.png)

Verificar Apache2:

```bash
apache2 -v
```

![](/home/manoel/bitsdeconhecimento/static/image4.png)

#### 3º Alguns pacotes úteis na hora de configurar que recomendo instalar:

```bash
apt install -y xz-utils bzip2 unzip curl snmp gzip
```

A partir deste ponto, as instruções podem ser encontradas no próprio site do Zabbix: [Download and install Zabbix](https://www.zabbix.com/download?zabbix=7.0). Segue o link para quem quiser baixar a versão mais atual do Zabbix, que no momento é a 7.0, e estamos usando o Debian 12. No próprio site do Zabbix, é possível escolher outras distribuições e versões, como servidores web (por exemplo, Nginx) ou outros bancos de dados, como o PostgreSQL, de acordo com sua preferência. Estou utilizando essas configurações por gosto pessoal, pois me sinto mais confortável para realizar troubleshooting. Vamos para o próximo passo, seguindo a documentação.

### I Instalando zabbix server

Instalando repositorio zabbix:

```bash
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.0+debian12_all.deb
```

```bash
dpkg -i zabbix-release_latest_7.0+debian12_all.deb
```

```bash
apt update
```

Agora, instalaremos os pacotes do Zabbix Server, Frontend e Agent.

```bash
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

Agora, é uma etapa muito importante, por isso deve ser feita com atenção e requer pequenas alterações nos comandos. Primeiro, vamos certificar-nos de que nosso banco de dados está rodando.

```bash
systemctl status maria.service
```

![](/home/manoel/bitsdeconhecimento/static/image5.png)

Vamos à configuração básica do MariaDB:

```bash
mysql_secure_installation
```

- Definir a senha do root (lembrando que esta será a senha solicitada toda vez para logar no console como root do MariaDB).
- Remover usuários anônimos.
- Desabilitar o login remoto para o usuário root.
- Remover o banco de dados de teste e o acesso a ele.

```bash
mysql -u root -p
```

Digite a senha do root criada anteriormente.

Agora, execute um comando por vez e com atenção. Recomendo colar os comandos no bloco de notas antes e fazer alterações, caso necessário.

```sql
create database zabbix character set utf8mb4 collate utf8mb4_bin;
```

No próximo passo, troque a senha "password" por uma de sua preferência, de maneira que seja segura e fácil de lembrar, ou salve-a em um local seguro. Assim, poderemos configurar o acesso nos próximos passos. (Deixarei "password" no tutorial para facilitar, ok?)

```sql
create user zabbix@localhost identified by 'password';
```

```sql
grant all privileges on zabbix.* to zabbix@localhost;
```

```
set global log_bin_trust_function_creators = 1;
```

```sql
quit;
```

Se tudo estiver certo, as saídas devem estar semelhantes a estas: 

![](/home/manoel/bitsdeconhecimento/static/image6.png)

Agora, importaremos o schema do banco, assim como os dados iniciais para o Zabbix Server. A senha que será solicitada é a senha que você informou no comando SQL, a qual recomendei mudar. Não é a senha root do banco.

```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

Desabilite a opção `log_bin_trust_function_creators` após importar o esquema do banco de dados.

```sql
mysql -uroot -p
```

```sql
set global log_bin_trust_function_creators = 0;
```

```sql
quit;
```

Fazendo isso, estamos quase no final desta primeira fase da instalação. Agora, vamos fazer algumas configurações no arquivo *zabbix_server.conf*, que é responsável por várias configurações do Zabbix.

Nossa primeira alteração é inserir a senha do banco no arquivo de configuração.

```bash
nano /etc/zabbix/zabbix_server.conf
```

Procure a linha *DBPassword* e coloque a senha definida na hora da criação do banco. Você pode usar `Ctrl + W` para pesquisar por *DBPassword* e alterar o arquivo.

![](/home/manoel/bitsdeconhecimento/static/image7.png)

Já podemos sair, salvar as novas configurações e reiniciar os serviços do Zabbix Server, Apache e Agent.

```bash
systemctl restart zabbix-server zabbix-agent apache2
```

E que esses serviços sejam inicializados após qualquer reboot do sistema.

```bash
systemctl enable zabbix-server zabbix-agent apache2
```

### Finalizando a instalação pela interface Web

Para acessar a interface web, vá para *[http://ENDEREÇO_IP/zabbix](http://ENDERE%C3%87O_IP/zabbix)*. Você pode verificar o endereço IP usando o comando:

```bash
hostname -I
```

![](/home/manoel/bitsdeconhecimento/static/image9.png)

Agora, já temos a nossa interface web para finalizar a instalação do Zabbix. No próximo passo, basta inserir a senha definida no banco e continuar. 

![](/home/manoel/bitsdeconhecimento/static/image10.png)

Neste passo, você definirá o nome do seu servidor, o fuso horário e os temas. Lembre-se de ajustar o fuso horário corretamente. 

![](/home/manoel/bitsdeconhecimento/static/image11.png)

Ele trará um resumo das configurações. Basta seguir, e ele informará que o Zabbix foi instalado com sucesso e que o arquivo de configuração `.php` foi criado corretamente.

Agora, chegamos à tela inicial de login: 

![](/home/manoel/bitsdeconhecimento/static/image13.png)

Usuário: Admin (lembre-se, a senha é "A" maiúscula e não minúscula).

Senha: zabbix

Esta é a primeira tela do Zabbix. Recomendo que você já troque a senha inicial do Zabbix.

![](/home/manoel/bitsdeconhecimento/static/image14.png)

No próximo post, instalaremos o Grafana, faremos a integração entre os dois e mostraremos como essa combinação é campeã no monitoramento.

E como diria Gandalf: "O que você faz com o tempo que tem?" Bem, nós estamos usando o nosso tempo para melhorar o monitoramento com Zabbix e Grafana!
