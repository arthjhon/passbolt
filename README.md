# Guia Completo: Instalação do Passbolt CE no Debian

Este repositório contém um guia passo a passo detalhado para instalar e configurar o Passbolt CE (Community Edition) em um servidor com Debian 11 ou 12. O guia cobre desde a preparação inicial do servidor até a solução de problemas comuns.

**Tecnologias utilizadas:**
* **Passbolt CE** (Gerenciador de Senhas Open Source)
* **Debian 11/12** (Sistema Operacional)
* **Nginx** (Servidor Web)
* **MariaDB** (Banco de Dados)
* **Certbot (Let's Encrypt)** (Certificados SSL/TLS Gratuitos)

---

## Índice

1.  [Pré-requisitos](#-pré-requisitos)
2.  [Passo 1: Preparação do Servidor](#-passo-1-preparação-do-servidor)
3.  [Passo 2: Instalação do Passbolt](#-passo-2-instalação-do-passbolt)
4.  [Passo 3: Configuração Final via Web](#-passo-3-configuração-final-via-web)
5.  [Manutenção e Comandos Úteis](#-manutenção-e-comandos-úteis)
    * [Atualizando o Passbolt](#atualizando-o-passbolt)
    * [Gerenciando o Certificado SSL](#gerenciando-o-certificado-ssl-com-certbot)
    * [Realizando Backups](#realizando-backups)
6.  [Solução de Problemas (Troubleshooting)](#-solução-de-problemas-troubleshooting)
    * [Erro: `E: Sub-process /usr/bin/dpkg returned an error code (1)`](#erro-e-sub-process-usrbindpkg-returned-an-error-code-1)
    * [Esqueci a Senha do Banco de Dados](#esqueci-a-senha-do-banco-de-dados)
    * [Permitir Acesso Remoto ao Banco de Dados](#permitir-acesso-remoto-ao-banco-de-dados)

---

## 📋 Pré-requisitos

Antes de começar, garanta que você possui:

* **Um VPS com Debian 11 (Bullseye) ou 12 (Bookworm)** recém-instalado. (Mínimo: 1 vCPU, 512MB RAM).
* **Acesso root** ou um usuário com privilégios `sudo`.
* **Um nome de domínio (ou subdomínio)**, por exemplo `passbolt.seusite.com.br`.
* **DNS Configurado:** Um registro DNS do tipo `A` apontando seu domínio para o IP público do VPS.
* **Credenciais de um servidor de e-mail (SMTP)** para que o Passbolt possa enviar convites e notificações.

---

## 🚀 Passo 1: Preparação do Servidor

Conecte-se ao seu servidor via SSH e atualize os pacotes do sistema.

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🔧 Passo 2: Instalação do Passbolt

Usaremos o script oficial para configurar o repositório e instalar o Passbolt junto com todas as suas dependências (Nginx, MariaDB, PHP, Certbot).

1.  **Baixe e execute o script de instalação do repositório:**
    ```bash
    wget [https://download.passbolt.com/ce/installer/passbolt-repo-setup.ce.sh](https://download.passbolt.com/ce/installer/passbolt-repo-setup.ce.sh)
    sudo bash ./passbolt-repo-setup.ce.sh
    ```

2.  **Instale o servidor Passbolt:**
    Este comando iniciará um assistente de configuração no seu terminal. Leia cada etapa com atenção.
    ```bash
    sudo apt install passbolt-ce-server
    ```

3.  **Siga o assistente de configuração:**
    * **MariaDB (Banco de Dados):**
        * Escolha `Yes` para configurar o banco de dados.
        * Defina uma senha segura para o usuário `root` do MariaDB.
        * Defina uma senha segura para o usuário do banco de dados do Passbolt (`passboltuser`).
    * **GPG Key:**
        * Escolha `Yes` para gerar uma nova chave GPG para o servidor.
    * **Nginx & Let's Encrypt (SSL):**
        * Escolha `Yes` para configurar o Nginx.
        * Digite seu nome de domínio completo (ex: `passbolt.seusite.com.br`).
        * Escolha `Yes` para usar o Let's Encrypt para gerar um certificado SSL gratuito. (Isso só funcionará se o seu DNS já estiver apontado corretamente).
    * **SMTP (E-mail):**
        * Preencha os dados do seu servidor de e-mail para que as notificações funcionem.

---

## 💻 Passo 3: Configuração Final via Web

1.  Após a instalação no terminal, acesse seu domínio em um navegador: `https://passbolt.seusite.com.br`.
2.  Siga os passos na interface web para criar o primeiro usuário administrador.
3.  **Faça o download da sua chave privada de segurança.** Guarde este arquivo em um local extremamente seguro. Sem ele, você perde o acesso à sua conta.
4.  Defina sua senha mestra.
5.  Instale a extensão do Passbolt para o seu navegador (Chrome, Firefox, Edge).
6.  Configure a extensão usando a chave privada que você baixou e sua senha mestra.

**Parabéns! Sua instância do Passbolt está pronta para uso.**

---

## ⚙️ Manutenção e Comandos Úteis

### Atualizando o Passbolt
As atualizações são distribuídas pelo `apt`. Para atualizar, basta rodar os comandos padrão do sistema:
```bash
sudo apt update
sudo apt upgrade
```

### Gerenciando o Certificado SSL com Certbot
O instalador já configura a renovação automática.

* **Testar a renovação (simulação):**
    ```bash
    sudo certbot renew --dry-run
    ```
* **Forçar a renovação:**
    ```bash
    sudo certbot renew
    ```

### Realizando Backups
**ESSA É A PARTE MAIS IMPORTANTE.** Um backup do Passbolt consiste em:
1.  A chave GPG privada do servidor (localizada em `/etc/passbolt/gpg/serverkey_private.asc`).
2.  Um dump do banco de dados.

Consulte a [documentação oficial de backup](https://help.passbolt.com/hosting/backup/ce.html) para o procedimento correto.

---

## 🛠️ Solução de Problemas (Troubleshooting)

### Erro: `E: Sub-process /usr/bin/dpkg returned an error code (1)`
Este é um erro genérico. Tente os seguintes passos, em ordem:

1.  **Corrigir pacotes quebrados:**
    ```bash
    sudo apt --fix-broken install
    ```
2.  **Reconfigurar pacotes pendentes:**
    ```bash
    sudo dpkg --configure -a
    ```
3.  **Remover completamente e reinstalar:**
    ```bash
    sudo apt purge passbolt-ce-server
    sudo apt clean
    sudo apt update
    sudo apt install passbolt-ce-server
    ```
4.  **Verifique a mensagem de erro real:** Role para cima no terminal e procure a mensagem de erro específica que ocorreu *antes* da linha `dpkg error code (1)`.

### Esqueci a Senha do Banco de Dados
Se você esqueceu a senha do usuário `root` do MariaDB ou do usuário `passboltuser`:

1.  **Pare o MariaDB e reinicie em modo de segurança:**
    ```bash
    sudo systemctl stop mariadb
    sudo mysqld_safe --skip-grant-tables &
    ```
2.  **Conecte-se sem senha e redefina a senha root:**
    ```bash
    sudo mysql -u root
    ```
    ```sql
    FLUSH PRIVILEGES;
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'SUA_NOVA_SENHA_FORTE';
    EXIT;
    ```
3.  **Reinicie o serviço normalmente:**
    ```bash
    sudo kill `cat /var/run/mariadb/mariadb.pid`
    sudo systemctl start mariadb
    ```
4.  **Redefina a senha do usuário do Passbolt** (descubra o nome de usuário em `/etc/passbolt/passbolt.php`):
    ```bash
    mysql -u root -p
    ```
    ```sql
    ALTER USER 'passboltuser'@'localhost' IDENTIFIED BY 'NOVA_SENHA_PARA_O_PASSBOLT';
    FLUSH PRIVILEGES;
    EXIT;
    ```
5.  **Atualize a nova senha** no arquivo `/etc/passbolt/passbolt.php` e reinicie o Nginx e o PHP.

### Permitir Acesso Remoto ao Banco de Dados
Para conectar ao banco de dados de um IP externo (ex: para usar um cliente de BD):

1.  **Altere o `bind-address`:**
    * Edite o arquivo `/etc/mysql/mariadb.conf.d/50-server.cnf`.
    * Mude `bind-address = 127.0.0.1` para `bind-address = 0.0.0.0`.
    * Reinicie o MariaDB: `sudo systemctl restart mariadb`.

2.  **Configure o Firewall (UFW):**
    * Libere o acesso à porta `3306` **apenas para o seu IP**:
    ```bash
    sudo ufw allow from SEU_IP_DE_ACESSO to any port 3306
    ```

3.  **Altere o `host` do usuário do banco de dados:**
    * Conecte-se ao MariaDB como root.
    * Renomeie o usuário para permitir acesso do seu IP (recomendado):
    ```sql
    RENAME USER 'passboltuser'@'localhost' TO 'passboltuser'@'SEU_IP_DE_ACESSO';
    FLUSH PRIVILEGES;
    ```
