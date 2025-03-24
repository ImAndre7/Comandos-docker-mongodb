# Configuração de um Cluster MongoDB com Docker

Este guia detalha os passos para configurar um cluster MongoDB com 4 nós usando Docker.

## Pré-requisitos

* Docker Desktop instalado.

## Passos de Configuração

### 1. Configurando o MongoDB no Docker Desktop

Primeiro, baixamos a imagem mais recente do MongoDB e iniciamos um contêiner para teste inicial.

docker pull mongodb/mongodb-community-server:latest
docker run --name mongodb -p 27017:27017 -d mongodb/mongodb-community-server:latest

### 2. Criando a Rede Docker
docker network create mongoCluster

### 3. Criando e Configurando os 4 Nós do Cluster
docker run -d --rm -p 27017:27017 --name mongo10 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo10
docker run -d --rm -p 27018:27017 --name mongo20 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo20
docker run -d --rm -p 27019:27017 --name mongo30 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo30
docker run -d --rm -p 27020:27017 --name mongo40 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo40

### 4. Configurar a inicialização do conjunto de réplicas
docker exec -it mongo10 mongosh

Salve a string "Connection To" em um bloco de notas para que voce possa utilizar depois, na string de conexão do MongoDB Compass: "mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.4.2"

### 5. Verificação do contender 
db.runCommand ({hello:1})

### 6. Inicializando o Conjunto de Réplicas
rs.initiate ({ _id: "myReplicaSet", members:[{_id:1, host: "mongo10"}, {_id:2, host: "mongo20"}, {_id:3, host: "mongo30"}, {_id:4, host: "mongo40"}]})
Após realizar o comando de "exit" para sair no terminal Shell

### 7. Verificando o Status do Conjunto de Réplicas
docker exec -it mongo10 mongosh --eval "rs.status()"

### 8. Conectando com o MongoDB Compass
mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.4.2

### 7. Testes e Verificações (MongoDB Compass já conectado)
rs.isMaster().primary

# Inserindo Dados no Nó Primário (MongoDB)
1- use CorporeSystem;
db.cliente.insertOne({ codigo: 1, nome: "Abacaxi" });
db.cliente.insertOne({ codigo: 2, nome: "Mamão" });
db.cliente.insertOne({ codigo: 3, nome: "Uva" });
db.cliente.insertOne({ codigo: 4, nome: "Maçã" });
db.cliente.insertOne({ codigo: 5, nome: "Laranja" });
2- db.cliente.find();

### 9. Encerrando o Nó Secundário e Realizar verificação. Após isso, fazer teste na inserção de dados e realizar a recuperação do Nó
docker stop mongo30
1- db.cliente.insertOne({ codigo: 6, nome: "Morango" });
2- db.cliente.find();
3- - docker run -d --rm -p 27019:27017 --name mongo30 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo30

### 10. Testando a Recuperação do Nó Primário

# Pare o nó primário
docker stop mongo10

# Inserir dados no nó primário antigo (deve falhar) (MongoDB)
db.cliente.insertOne({ codigo: 7, nome: "Tangerina" });

# Conecte-se ao novo nó primário (porta 27018) (MongoDB)
mongodb://127.0.0.1:27018/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.4.2

# Verifique o status do conjunto de réplicas e o novo nó primário
docker exec -it mongo20 mongosh --eval "rs.status()"
rs.isMaster().primary

# Inserindo e verificando Dados no Nó Primário (New) (MongoDB)
1- use CorporeSystem;
db.cliente.insertOne({ codigo: 1, nome: "Kiwi" });
db.cliente.insertOne({ codigo: 2, nome: "Banana" });
2- db.cliente.find();
