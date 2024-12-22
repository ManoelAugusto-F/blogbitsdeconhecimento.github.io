---
---
title: "Configurando IP Estático no Debian"
date: 2024-07-07
draft: false
categories:
  - Tutorial
author: "Manoel Augusto"
tags:
  - Linux
  - Debian
  - Redes
description: "Aprenda a configurar um IP estático em uma interface de rede no Debian."
---


---

# Configurando IP Estático no Debian

Neste tutorial, aprenderemos a configurar uma interface de rede para utilizar um IP estático no Debian.

## Quando é necessário um IP Estático?

Um IP estático é essencial em situações como:

- Servidores Web
- Bancos de dados
- Serviços de NAS
- Qualquer serviço que precise de um IP fixo

Se o host estiver configurado para DHCP, o IP pode mudar após reinicializações ou expiração do IP.

## Configuração do IP Estático

1. **Identifique o Nome da Interface de Rede**
   
   Utilize o comando a seguir para descobrir o nome da interface de rede:
   
   ```bash
   ip addr show
   ```
   
   A saída será semelhante a:
   
   ```bash
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host
          valid_lft forever preferred_lft forever
   2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
       link/ether 00:e0:4c:36:59:04 brd ff:ff:ff:ff:ff:ff
       inet 192.168.1.100/24 brd 192.168.1.255 scope global eth0
          valid_lft forever preferred_lft forever
       inet6 fe80::20e:4cff:fe36:5904/64 scope link
          valid_lft forever preferred_lft forever
   ```
   
   No exemplo, o nome da interface é `eth0`. Esse nome pode variar, então ajuste conforme necessário.

2. **Edite o Arquivo de Configuração**
   
   Abra o arquivo de configuração das interfaces:
   
   ```bash
   sudo nano /etc/network/interfaces
   ```
   
   O conteúdo padrão será algo como:
   
   ```bash
   # Este arquivo descreve as interfaces de rede disponíveis no seu sistema
   # e como ativá-las. Para mais informações, consulte interfaces(5).
   
   # A interface de loopback
   auto lo
   iface lo inet loopback
   
   # A interface de rede primária
   allow-hotplug eth0
   iface eth0 inet dhcp
   ```
   
   Modifique o arquivo para definir um IP estático:
   
   ```bash
   # Endereço IP estático
   auto eth0
   iface eth0 inet static
       address 192.168.1.100
       netmask 255.255.255.0
       network 192.168.1.0
       broadcast 192.168.1.255
       gateway 192.168.1.1
   ```
   
   Lembre-se de substituir `eth0` pelo nome correto da sua interface.

3. **Reinicie o Serviço de Rede**
   
   Após salvar as alterações, reinicie o serviço de rede:
   
   ```bash
   sudo systemctl restart networking
   ```
   
   Verifique se o serviço está ativo:
   
   ```bash
   sudo systemctl status networking
   ```
   
   A saída deve ser similar a:
   
   ```bash
   ● networking.service - LSB: Bring up/down networking
      Loaded: loaded (/etc/init.d/network; generated)
      Active: active (exited) since Thu 2024-04-25 12:00:00 UTC; 5min ago
        Docs: man:systemd-sysv-generator(8)
       Tasks: 0 (limit: 32768)
      Memory: 0B
      CGroup: /system.slice/network.service
   
   Apr 25 12:00:00 debian systemd[1]: Starting LSB: Bring up/down networking...
   Apr 25 12:00:00 debian network[1234]: Bringing up loopback interface:  [  OK  ]
   Apr 25 12:00:00 debian network[1234]: Bringing up interface eth0:  [  OK  ]
   Apr 25 12:00:00 debian systemd[1]: Started LSB: Bring up/down networking.
   ```

Essas instruções são aplicáveis tanto para servidores Linux baseados em Debian quanto para desktops pessoais.
