# Deploy Manual de Wordpress em EC2, com domínio e HTTPS

## Identificar etapas, recursos e serviços necessários.

1. Criar uma instância nova, Ubuntu 22.04 - Chamada Wordpress, liberando as portas 22, 80 e 443. (Use uma chave já existente)
2. Servidor Web - Apache
3. Linguagem de Programação - PHP
4. Banco de Dados - MySQL
5. Baixar o Wordpress
6. Efetuar a instalação e configuração dentro da instância
7. Efetuando a instalação e configuração do NOIP (Ter uma conta)
8. Configurando e gerando um certificado auto-assinado (HTTPs)

## Execução

### Atualizando repositórios e a instância

```bash
sudo apt update && sudo apt upgrade -y
```

### Instalando apache

```bash
sudo apt install apache2
```

### Instalando toda base PHP

```bash
sudo apt install php8.1 libapache2-mod-php8.1 php8.1-mysql php8.1-common php8.1-curl php8.1-xml php8.1-mbstring php8.1-gettext php8.1-pdo php8.1-gd php8.1-zip php8.1-soap php8.1-xmlrpc php8.1-intl php8.1-mysqlnd php8.1-cli php8.1-dev php8.1-zip php8.1-bz2 php-pear
```

### Instalando MySQL

```bash
sudo apt install mysql-server-8.0
```

### Configurando o MySQL

```bash
sudo mysql -u root
```

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'senha_da_nasa';
CREATE DATABASE WPDEPLOY;
CREATE USER 'wp-user'@'localhost' IDENTIFIED BY 'senha_da_nasa';
GRANT ALL ON WPDEPLOY.* TO 'wp-user'@'localhost' WITH GRANT OPTION;
```

### Baixando, ajustando e instalando o Wordpress

Acesse o site do [Wordpress](https://br.wordpress.org) pegue o link do último pacote do wordpress.

Logo após ter o link, execute a sequência abaixo.

```bash
cd # Volta para pasta padrão do usuario ubuntu
wget https://br.wordpress.org/latest-pt_BR.zip #Baixa a ultima versão do Wordpress

sudo apt install unzip
unzip latest-pt_BR.zip # Descompacta o Wordpress

sudo mv wordpress /var/www/html/ # Move para a pasta /var/www/html
cd /var/www/html # Acessa a pasta /var/www/html
sudo chown www-data:www-data -R wordpress # Altera o dono e o grupo da pasta para www-data

# Altere Documentroot para novo caminho
sudo vim /etc/apache2/sites-available/000-default.conf

sudo service apache2 restart 
```

### Baixando, instalando e configurado o NOIP [Não esqueça de criar uma conta antes]

```bash
wget --content-disposition <https://www.noip.com/download/linux/latest> # Baixa o DUC 3
tar xf noip-duc_3.3.0.tar.gz 
cd /home/$USER/noip-duc_3.3.0/binaries && sudo apt install ./noip-duc_3.3.0_amd64.deb 
noip-duc -g <seudominio> -u <seu-usuario> -p <sua-senha>

# Copia arquivo de configuração da inicialização automática
sudo cp noip-duc_3.3.0/debian/service /etc/systemd/system/noip-duc.service

# Cria arquivo de configuração permanente
sudo vim /etc/default/noip-duc # Colar dentro do arquivo a estrutura abaixo
  ## File: /etc/default/noip-duc
  NOIP_USERNAME=
  NOIP_PASSWORD=
  NOIP_HOSTNAMES=

# Executar após criar os arquivos acima  
sudo systemctl daemon-reload
sudo systemctl enable noip-duc
sudo systemctl start noip-duc
```

### Referência direto do NOIP

🔗 https://www.noip.com/support/knowledgebase/install-linux-3-x-dynamic-update-client-duc

🔗 https://www.noip.com/support/knowledgebase/running-linux-duc-v3-0-startup-2

------

## Gerando e configurando certificado auto assinado - HTTPS

```bash
sudo apachectl -M | grep ssl # Verifica se o módulo SSL está ativo no Apache, se não der nenhum retorno o módulo não está ativo.
sudo a2enmod ssl # Ativa módulo SSL
sudo systemctl restart apache2.service
sudo apt install certbot python3-certbot-apache
sudo certbot --apache -d <seudominio>
```