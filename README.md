#Instalação e Configuração do OpenCMS com PostgreSQL e NGINX

#Visão Geral

Nesse projeto, configurei o OpenCMS usando PostgreSQL como banco de dados e configurei o NGINX como proxy reverso para o Tomcat9. Além disso, ajustei a URL para que o OpenCMS pudesse ser acessado de forma amigável via opencms.local. Aqui estão os detalhes e o passo a passo do que eu fiz.


#1. Configuração do PostgreSQL

Passo 1: Verifiquei se o PostgreSQL já estava em execução

  Primeiro, eu verifiquei se o PostgreSQL estava rodando:
  sudo systemctl status postgresql
  
  Se o PostgreSQL não estivesse rodando, eu teria instalado com:
  sudo apt update
  sudo apt install postgresql postgresql-contrib
  
  
  Passo 2: Criação do Banco de Dados e Usuário
  Acessei o PostgreSQL com o usuário postgres:
  sudo -u postgres psql
  
  Em seguida, criei o banco de dados opencms e o usuário opencmsuser:
  CREATE DATABASE opencms;
  CREATE USER opencmsuser WITH PASSWORD 'password';
  GRANT ALL PRIVILEGES ON DATABASE opencms TO opencmsuser;
  \q
  
  Decisão: O banco opencms foi criado para armazenar os dados da aplicação, enquanto o usuário opencmsuser foi criado para isolar o acesso ao banco, oferecendo uma camada adicional de segurança e organização.


#2. Configuração do Tomcat9 e OpenCMS

  Passo 3: Baixar e Instalar o Tomcat9
  Use o comando wget para fazer o download do arquivo:
  wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.96/bin/apache-tomcat-9.0.96.tar.gz
  
  Após o download, extraia o arquivo para o diretório desejado. No meu caso, eu extrai para o diretório /opt/tomcat9:
  sudo mkdir -p /opt/tomcat9
  sudo tar -xvzf apache-tomcat-9.0.96.tar.gz -C /opt/tomcat9 --strip-components=1
  
  Altere as permissões para garantir que o Tomcat tenha as permissões corretas:
  sudo chmod +x /opt/tomcat9/bin/*.sh
  
  Inicie o Tomcat:
  sudo /opt/tomcat9/bin/startup.sh
  
  Passo 4: Baixar e Instalar o OpenCMS
  
  Baixei o OpenCMS diretamente do site oficial. Coloquei o arquivo WAR do OpenCMS na pasta do Tomcat, em /opt/tomcat9/webapps, com o seguinte comando:
  sudo mv ~/Downloads/opencms-18.0.war /opt/tomcat9/webapps/
  
  Descompactei o arquivo:
  unzip opencms-18.0.war -d /opt/tomcat9/webapps/opencms
  
  Passo 5: Configurar a Conexão do OpenCMS com o PostgreSQL
  
  Para conectar o OpenCMS ao PostgreSQL, editei o arquivo opencms.properties:
  sudo nano /opt/tomcat9/webapps/opencms/WEB-INF/config/opencms.properties
  
  Altere as seguintes linhas para configurar o PostgreSQL:
  db.default.driver=org.postgresql.Driver
  db.default.url=jdbc:postgresql://localhost:5432/opencms
  db.default.user=opencmsuser
  db.default.password=1234
  
  Isso garante que o OpenCMS utilize o PostgreSQL como banco de dados.
  
  Passo 6: Reiniciar o Tomcat9
  
  Após as alterações, reiniciei o Tomcat9 para que o OpenCMS reconhecesse as configurações:
  sudo systemctl restart tomcat9
  
  Passo 7: Finalizar a Instalação do OpenCMS
  
  Acessei o OpenCMS pelo Tomcat via navegador, utilizando o endereço:
  http://localhost:8080/opencms/setup
  
  Segui o assistente de instalação e configurei o PostgreSQL com as credenciais:
  
      Banco de dados: opencms
      Usuário: opencmsuser
      Senha: 1234
  
  Login do OpenCMS: Ao final da instalação, o login para o ambiente de trabalho do OpenCMS (workplace) foi:
  
      Usuário: Admin
      Senha: 1234

#3. Configuração do NGINX como Proxy Reverso

  Passo 8: Instalei o NGINX
  
  Eu já tinha o NGINX instalado, mas, caso não estivesse, poderia instalar com:
  sudo apt update
  sudo apt install nginx
  
  Passo 9: Configurar o Proxy Reverso no NGINX
  
  Para garantir que o NGINX atuasse como proxy reverso, editei o arquivo de configuração do NGINX:
  sudo nano /etc/nginx/sites-available/default
  
  Adicionei as seguintes linhas:
  server {
      listen 80;
      server_name opencms.local;
  
      location /opencms/ {
          proxy_pass http://localhost:8080/opencms/;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
  
      location / {
          return 301 /opencms/index;
      }
  }
  
  Decisão: Configurei o proxy_pass para redirecionar todo o tráfego da URL opencms.local para o OpenCMS rodando no Tomcat na porta 8080. A linha location / {return 301 /opencms/index;} foi adicionada para redirecionar automaticamente o tráfego para a pagina que criei utilizando jsp.
  
  Passo 10: Configurar o Arquivo /etc/hosts
  
  Adicionei uma entrada no arquivo /etc/hosts para que a URL opencms.local resolvesse corretamente:
  sudo nano /etc/hosts
  
  Adicionei a linha:
  127.0.0.1   opencms.local
  
  Passo 11: Testar e Reiniciar o NGINX
  
  Após a configuração, testei o NGINX para garantir que tudo estava correto:
  sudo nginx -t
  
  Como não houve erros, reiniciei o NGINX:
  sudo systemctl restart nginx
  
  Agora, o OpenCMS está acessível via http://opencms.local.


#Conclusão

  Esse foi o processo que segui para configurar o OpenCMS usando PostgreSQL e NGINX como proxy reverso. A documentação também inclui as credenciais de login (Admin / 1234) e as configurações necessárias para garantir que o OpenCMS funcione corretamente através do NGINX.
