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




