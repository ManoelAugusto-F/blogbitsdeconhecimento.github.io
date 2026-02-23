---
title: "Tutorial de como instalar e configurar o Zabbix Server e o Grafana Parte 2"
date: 2026-02-21T20:47:15-03:00
draft: false
categories:
  - Tutorial
author: "Manoel Augusto"
tags:
  - Zabbix
  - Grafana
  - Monitoramento
keywords:
  - Zabbix
  - Grafana
  - Monitoramento
---

Olá pessoal, há quanto tempo. Peço desculpa por esse sumiço de mais de um ano. Muitas coisas aconteceram, eu deixei de lado o blog. Conto um pouco mais no [Sobre](https://bitsdeconhecimento.blog.br/about/), mas enfim, voltamos para terminar esse tutorial de como instalar e configurar o Grafana. Vamos nessa?

## Pré-requisitos

* Um servidor Debian ou Ubuntu atualizado, com acesso à internet e com acesso root.

## Preparando o terreno

Primeiro, vamos garantir que seu sistema esteja atualizado:

```bash
sudo apt update && sudo apt upgrade -y
```

Após isso, vamos instalar alguns programas que vão nos ajudar no processo, que são necessários para baixar e verificar as chaves dos repositórios que utilizaremos:

```bash
sudo apt install -y software-properties-common curl gnupg2 apt-transport-https
```

E com isso terminamos a preparação do terreno.

## Instalação do Grafana

Por padrão, o Grafana não está nos repositórios do Debian, e por isso vamos pegar alguns comandos no site oficial do Grafana, que é [Grafana Labs](https://grafana.com/docs/grafana/latest/), e lá vamos ver essa tela:

![image01](/images/Grafana/image01.png)

Que dá direto na documentação e ajuda a gente a resolver problemas. Lembre-se: a documentação é nossa amiga. Está com problema? Vem na documentação. Aí vai me dizer que o ChatGPT sabe mais. Será que você vai aprender ou vai dar o Ctrl+C + Ctrl+V? Pense nisso e vamos lá. Já vou resumir aqui como você vai instalar sem precisar ler a documentação.

Primeiro, vamos criar uma pasta para receber a nossa chave GPG:

```bash
sudo mkdir -p /etc/apt/keyrings/
```

Agora, vamos fazer a importação dela:

```bash
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```

Agora adicionaremos o repositório de versões estáveis. Fique atento: se for usar a documentação, ela coloca os dois repositórios, beta e estáveis, um seguido do outro, e você pode acabar adicionando os dois.

```bash
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

E agora atualizaremos o repositório recém-adicionado:

```bash
sudo apt-get update
```

E agora, finalmente, instalamos o Grafana OSS:

```bash
sudo apt-get install grafana
```

Se tudo que você fez deu certo, a saída será igual à da imagem abaixo:

![image02](/images/Grafana/image02.png)

Agora, com o Grafana instalado, vamos para o próximo passo.

## Configurações iniciais

Primeiramente, precisamos iniciar o serviço e depois habilitar para que, toda vez que o servidor for iniciado, o serviço suba automaticamente.

O primeiro comando é iniciar o serviço do Grafana:

```bash
sudo systemctl start grafana-server
```

E depois usamos o comando que inicia automaticamente após um reboot:

```bash
sudo systemctl enable grafana-server
```

E agora conferimos se o serviço está ativo com:

```bash
sudo systemctl status grafana-server
```

Se ocorreu tudo bem, agora conseguimos acessar o Grafana via interface web. Abra seu navegador e acesse o endereço `http://<IP_DO_SEU_SERVIDOR_DEBIAN>:3000`. A porta padrão do Grafana é a 3000.

* Usuário: admin
* Senha: admin

![image03](/images/Grafana/image03.png)

Ao fazer o primeiro login, o sistema solicitará que você altere a senha para uma nova. Faça isso e guarde-a em um local seguro.

## Instalação e configuração do plugin do Zabbix

Por padrão, o Grafana não se comunica nativamente com o Zabbix e vice-versa, e é usando esse plugin que foi desenvolvido por [Alexander Zobnin](https://github.com/alexanderzobnin). Coloquei o GitHub dele, vale a pena dar uma olhada, mas vamos para o que interessa: instalar o plugin via linha de comando. O Grafana vem com uma ferramenta de linha de comando própria, o `grafana-cli`, para gerenciar plugins. Vamos usá-la.

```bash
sudo grafana-cli plugins install alexanderzobnin-zabbix-app
```

E se ocorreu tudo bem, vamos ter essa saída no terminal:

![image04](/images/Grafana/image04.png)

E agora vamos dar restart no Grafana para que entrem em vigência as mudanças que acabamos de fazer, e é sempre importante lembrar que, sempre que instalarmos plugins e algumas atualizações, é bom dar restart no Grafana:

```bash
sudo systemctl restart grafana-server
```

E agora que reiniciamos o server, vamos na interface gráfica para vermos o plugin instalado.

Na interface web, clique em **Connections** e depois em **Add new connection**, e vai aparecer uma infinidade de plugins. Na busca, você vai pesquisar por **Zabbix**, e deve aparecer a imagem abaixo:

![image05](/images/Grafana/image05.png)

Quando chegar nessa tela, é só clicar sobre o plugin, e teremos esse detalhe que explica sobre o plugin e dá a opção de **Enable**, que é ativar o plugin e conectar o Grafana e o Zabbix. Essa é a segunda tela após clicar no plugin:

![image06](/images/Grafana/image06.png)

Agora clicamos em **Enable** e vamos à parte mais importante, a configuração do seu plugin, que fornecerá os dados para criar os dashboards. Agora vamos clicar em **Connections** novamente e depois em **Data sources**, e pesquise por **Zabbix**. Agora clique na opção que vai aparecer e você terá essa tela:

![image07](/images/Grafana/image07.png)

E agora vamos preencher campo por campo:

1. **Nome (Name)**
   Zabbix (ou o nome que preferir)

2. **URL**
   [http://192.168.1.44/zabbix/api_jsonrpc.php](http://192.168.1.44/zabbix/api_jsonrpc.php) (IP do seu servidor)
   Fundamental que termine com `/api_jsonrpc.php`

3. **Access**
   Server (default)

4. **Authentication Method**
   Basic Auth (marque esta opção)

5. **Basic Auth Details**
   User: grafana (ou usuário que você criou, vendo no tutorial anterior que foi retificado, eu esqueci, sou humano)
   Password: (a senha do usuário informado)

6. **Zabbix API details** (role a página para baixo)
   Username: grafana (ou seu usuário do Zabbix)
   Password: (a senha do usuário informado)

7. **Trends (Configurações de desempenho – importante marcar)**
   Enable trends: SIM (marque a caixa)
   Trends from: 7d (7 dias)
   Trends range: 4d (4 dias)

8. **Botão final**
   Clique em **Save & Test**

Tudo ocorrendo bem, essa será a sua visão:

![image08](/images/Grafana/image08.png)


E agora você já tem acesso a todos os dados do Zabbix e pode começar a criação de dashboards. Quem sabe eu faça um tutorial de dashboards, é algo para o futuro. Obrigado pela paciência e por ter chegado até aqui, e até o próximo post.
