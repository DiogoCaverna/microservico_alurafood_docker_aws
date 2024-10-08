A equipe vai precisar analisar alternativas para disponibilizar e distribuir essa aplicação. 
Para rodar localmente o projeto, tivemos que instalar o banco de dados, o JDK, o Maven e o Intellij. 
Para migrar a estrutura para um ambiente cloud, a primeira alternativa é o Docker.

O Docker é uma ferramenta de conteinerização, que facilita o processo de entrega e deploy. 
Isso porque ele nos permite realizar um isolamento das aplicações. Vamos isolar as aplicações em containers e, 
com isso, obtemos mais flexibilidade para fazer alterações, atualizações e controle de versões.

Essa ferramenta também facilita a distribuição da aplicação independente do ambiente. 
Basta rodar alguns comandos, o container vai ser criado a partir de uma imagem e vai permitir a replicação desse ambiente.

Para recapitular e analisar como o Docker funciona, executaremos o banco de dados em um container e acessar a aplicação utilizando
o banco de dados do container. Com esse propósito, vamos parar o serviço do MySQL:

      sudo systemctl stop mysql

Verificando o status do serviço:

      sudo systemctl status mysql

Reiniciando o serviço:

      sudo systemctl start mysql

Para rodar o MySQL no docker:

      docker run -d -e MYSQL_ROOT_PASSWORD=************ -p 3306:3306 --name alurafood mysql:latest

_-d: O sinalizador -d (detached mode) faz com que o contêiner seja executado em segundo plano. 
Assim, você não precisa manter o terminal aberto para que o contêiner continue em execução._

_-e MYSQL_ROOT_PASSWORD=*************: Esta opção -e define uma variável de ambiente dentro do contêiner.
Neste caso, você está definindo a senha do usuário root do MySQL como ************._
 
_-p 3306:3306: O sinalizador -p mapeia uma porta no host (o sistema onde o Docker está rodando) para uma porta dentro do contêiner. 
Aqui, você está mapeando a porta 3306 do host (sua máquina local) para a porta 3306 do contêiner. 
Isso significa que qualquer conexão à porta 3306 do host será redirecionada para a porta 3306 no contêiner, que é a porta padrão do MySQL._



Conectar-se ao MySQL do seu host:

      mysql -h 127.0.0.1 -P 3306 -u root -p

### Para saber mais: [utilizando o MySQL com Docker](https://cursos.alura.com.br/extra/alura-mais/como-rodar-mysql-com-docker--c132). ###

## Gerando a imagem da aplicação ##

Para colocarmos a nossa aplicação de pedidos em um container, primeiro, é necessário criarmos uma imagem dela. Para isso, é preciso criarmos um arquivo chamado Dockerfile.

O Dockerfile é um arquivo texto que descreve alguns comandos e instruções, que servem para construirmos a imagem. A partir dessas orientações, subiremos o container.

Vamos criar o arquivo Dockerfile(sem extensão mesmo) na raiz do projeto. No site do [Spring](https://spring.io/guides/gs/spring-boot-docker), copiaremos o segundo exemplo para dentro do arquivo Dockerfile:

      FROM openjdk:17-jdk-alpine
      RUN addgroup -S spring && adduser -S spring -G spring
      USER spring:spring
      ARG JAR_FILE=target/*.jar
      COPY ${JAR_FILE} app.jar
      ENTRYPOINT ["java","-jar","/app.jar"]

Após colar esse código na pasta do Dockerfile, vamos analisar o trecho. No caso eu ja fiz a alteração da openjdk para 17.

### Explicação do Dockerfile: ###

      FROM openjdk:17-jdk-alpine: 
      Usa a imagem base do OpenJDK 17 com Alpine, uma versão leve do Linux.

      RUN addgroup -S spring && adduser -S spring -G spring: 
      Cria um grupo e um usuário chamado spring, sem senha (-S), para rodar o contêiner de forma segura.
      
      USER spring:spring: 
      Define que o contêiner será executado com o usuário spring e seu grupo correspondente.
      
      ARG JAR_FILE=target/*.jar: 
      Define uma variável de build chamada JAR_FILE, que será usada para especificar o caminho do arquivo .jar a ser copiado.
      
      COPY ${JAR_FILE} app.jar: 
      Copia o arquivo .jar gerado para dentro do contêiner, nomeando-o como app.jar.
      
      ENTRYPOINT ["java","-jar","/app.jar"]: 
      Define o comando padrão para executar o contêiner, iniciando a aplicação Java com o arquivo app.jar.

### Para construir uma imagem Docker a partir do seu Dockerfile, siga os seguintes passos: ###

### *1. Verifique se você está no diretório correto:* ###
Certifique-se de que o Dockerfile e o arquivo .jar gerado estão no mesmo diretório ou no caminho correto.

### *2. Gere o arquivo .jar (se ainda não tiver sido gerado):* ###
Se você estiver usando o Maven, por exemplo, execute o comando abaixo para gerar o arquivo .jar no diretório target:

      mvn clean package

### *3. Execute o comando docker build:* ###
Agora, construa a imagem Docker usando o seguinte comando:

      docker build -t nome-da-imagem .

*Explicação do comando docker build:*

       *A flag -t define o nome da imagem. No nosso caso, a imagem será chamada pedidos-ms.
       
       *O ponto (.) indica que o contexto de build é o diretório atual, ou seja, o diretório onde o Dockerfile está localizado.


### *4.Verifique se a imagem foi criada e execute o contêiner:* ###

      docker images 

      docker run -d -p 8080:8080 nome-da-imagem

*(-d) inicia o contêiner em segundo plano* 

*(-p) mapeia a porta 8080 do host para a porta 8080 do contêiner.*

## Passos para enviar uma imagem para o Docker Hub: ##

### *1. Criar uma conta no Docker Hub:* ###

Se ainda não tem uma conta, crie uma em [hub.docker.com.](hub.docker.com)

Uma vez que você tenha uma conta, você pode criar repositórios públicos diretamente na interface do Docker Hub.

### *2. Login no Docker Hub via CLI:* ###

No terminal, faça login com suas credenciais do Docker Hub usando o seguinte comando:

      docker login

### *3. Taguear a imagem:* ###

Antes de enviar uma imagem, é necessário "tagueá-la" corretamente para associá-la ao seu repositório no Docker Hub. O formato da tag é:

      docker tag <image-id> <docker-hub-username>/<repository-name>:<tag>

Se, por exemplo, sua imagem foi criada com o comando anterior e você tem o nome de usuário meuusuario, e quer usar o repositório meuapp com a tag latest, o comando seria assim:

      docker tag <image-id> meuusuario/meuapp:latest

### *4. Enviar a imagem para o Docker Hub:* ###

Com a imagem corretamente tagueada, você pode enviá-la ao Docker Hub com o comando:

      docker push meuusuario/meuapp:latest

### Exemplo completo: ###
*Aqui está um exemplo completo do processo:*

      # 1. Fazer login no Docker Hub
      docker login

      # 2. Listar as imagens para encontrar o ID
      docker images

      # 3. Taguear a imagem para o Docker Hub
      docker tag <image-id> meuusuario/meuapp:latest

      # 4. Fazer o push da imagem
      docker push meuusuario/meuapp:latest

Agora a imagem estará disponível publicamente para que qualquer pessoa possa baixá-la usando:

      docker pull meuusuario/meuapp:latest

### Para saber mais sobre o Dockerfile: [Desvendando o DockerFile](https://www.alura.com.br/artigos/desvendando-o-dockerfile) ###



# Migrando para a Cloud #

O Cloudformation é um serviço da AWS, em que temos um arquivo no formato de JSON ou YML, que possui as etapas para subir a infraestrutura e provisionar o recurso. Há também, a opção do Terraform, a mesma lógica do Cloudformation, mas, com a linguagem declarativa disponível para diversas plataformas, além da AWS.

No nosso projeto, usaremos:

-AWS

-AWS Fargate

-Cloud Development Kit com Java

Inicialmente, para preparar o ambiente, precisamos instalar e configurar o AWS CLI, sigla para AWS Command Line Interface, usando as suas credenciais(*AWS Access Key ID e AWS Secret Access Key*).

[Instalar ou atualizar a versão mais recente da AWS CLI](https://docs.aws.amazon.com/pt_br/cli/latest/userguide/getting-started-install.html.)

*Verificar a versão da CLI instalada:*

      aws --version

*É necessário configurar com a sua credencial, para ser possível executar o deploy:*

[Configurar a AWS CLI](https://docs.aws.amazon.com/pt_br/cli/latest/userguide/cli-chap-configure.html)

*Para configurar a AWS CLI, basta digitar o comando:*

      aws configure

É preciso realizar a configuração do [AWS Cloud Development Kit](https://aws.amazon.com/pt/cdk/) 

[Comece a usar o AWS CDK: GUIA DE CONCEITOS BÁSICOS](https://aws.amazon.com/pt/getting-started/guides/setup-cdk/)

instale o CDK através do comando npm install -g aws-cdk. Novamente, será necessário verificar a versão:

      cdk --version

Vamos para o [Módulo 3: Criar o primeiro projeto do AWS CDK do site](https://aws.amazon.com/pt/getting-started/guides/setup-cdk/module-three/). Definiremos a nossa infraestrutura em Java e criaremos o pacote que conterá o projeto.

*No terminal, criaremos uma pasta para o nosso projeto:*

      mkdir alura-aws-infra

*Dentro da pasta digitaremos o comando:*

      cdk init --language java


 











