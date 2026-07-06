# Projeto Final Docker — Wiki.js com PostgreSQL

## 1. Identificação do Projeto

**Aluno:** Maciel  
**Disciplina:** Tópicos Avançados em TI  
**Projeto:** Implantação de Infraestrutura com Docker Compose  
**Aplicação:** Wiki.js  
**Banco de dados:** PostgreSQL  

Este projeto tem como objetivo implantar uma infraestrutura conteinerizada utilizando Docker Compose para executar o Wiki.js com banco de dados PostgreSQL.

A proposta não envolve o desenvolvimento de uma aplicação do zero, mas sim a configuração de um ambiente funcional, seguro, persistente e resiliente, seguindo boas práticas de infraestrutura com containers.

---

## 2. Objetivo

O objetivo principal deste projeto é configurar uma infraestrutura em duas camadas utilizando Docker Compose:

- Camada de aplicação: Wiki.js
- Camada de dados: PostgreSQL

A solução foi projetada para atender aos seguintes requisitos técnicos:

- Uso de Docker Compose;
- Separação entre aplicação e banco de dados;
- Banco de dados sem exposição externa da porta 5432;
- Exposição externa apenas da aplicação Wiki.js pela porta 3000;
- Uso de rede interna Docker;
- Persistência de dados com volumes nomeados;
- Uso de variáveis de ambiente com arquivo `.env`;
- Entrega de um `.env.example` como modelo;
- Uso de `.gitignore` para impedir o versionamento do `.env`;
- Healthcheck no PostgreSQL com `pg_isready`;
- Inicialização do Wiki.js somente após o banco estar saudável.

---

## 3. Arquitetura da Solução

A arquitetura utilizada é composta por duas camadas:

```text
Usuário / Navegador
        |
        | localhost:3000
        |
     Wiki.js
        |
        | Rede interna Docker: wiki-network
        |
   PostgreSQL

   A aplicação Wiki.js é a única camada acessível externamente pelo navegador, através da porta 3000.

O banco de dados PostgreSQL não expõe a porta 5432 para a máquina host. A comunicação entre Wiki.js e PostgreSQL ocorre exclusivamente dentro da rede interna do Docker.

4. Tecnologias Utilizadas
Docker
Docker Compose
Wiki.js
PostgreSQL
PowerShell
Git/GitHub
5. Estrutura do Projeto
projeto_final_docker/
│
├── docker-compose.yml
├── .env.example
├── .gitignore
└── README.md
Descrição dos arquivos
Arquivo	Função
docker-compose.yml	Arquivo principal de orquestração dos containers
.env.example	Modelo das variáveis de ambiente necessárias
.gitignore	Impede o envio do arquivo .env real para o GitHub
README.md	Documentação técnica do projeto

O arquivo .env existe apenas localmente e não deve ser enviado ao repositório, pois contém informações sensíveis como usuário e senha do banco de dados.

6. Configuração das Variáveis de Ambiente

O projeto utiliza variáveis de ambiente para evitar hardcoding de credenciais no docker-compose.yml.

O arquivo .env.example serve como modelo:

POSTGRES_DB=wikijs
POSTGRES_USER=seu_usuario
POSTGRES_PASSWORD=sua_senha_segura

DB_TYPE=postgres
DB_HOST=db
DB_PORT=5432
DB_USER=seu_usuario
DB_PASS=sua_senha_segura
DB_NAME=wikijs

Para executar o projeto localmente, é necessário criar um arquivo .env com base no .env.example.

Exemplo de .env local:

POSTGRES_DB=wikijs
POSTGRES_USER=wikijs_user
POSTGRES_PASSWORD=senha_forte_123

DB_TYPE=postgres
DB_HOST=db
DB_PORT=5432
DB_USER=wikijs_user
DB_PASS=senha_forte_123
DB_NAME=wikijs

O arquivo .env está listado no .gitignore para evitar que credenciais reais sejam versionadas.

7. Docker Compose

O arquivo docker-compose.yml define dois serviços principais:

db: banco de dados PostgreSQL;
wiki: aplicação Wiki.js.
Serviço do Banco de Dados

O serviço db utiliza a imagem:

image: postgres:16

As credenciais do banco são carregadas por variáveis de ambiente:

environment:
  POSTGRES_DB: ${POSTGRES_DB}
  POSTGRES_USER: ${POSTGRES_USER}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

Essa abordagem evita que senhas e usuários fiquem escritos diretamente no docker-compose.yml.

8. Segurança e Isolamento do Banco

O banco PostgreSQL não possui a diretiva ports.

Isso significa que a porta 5432 não é exposta para a máquina host.

Exemplo do que não foi utilizado:

ports:
  - "5432:5432"

A ausência desse mapeamento é proposital. O PostgreSQL não precisa ser acessado diretamente pelo navegador ou externamente. Apenas o container do Wiki.js precisa se comunicar com ele.

A comunicação ocorre internamente pela rede Docker wiki-network.

Na prática, ao executar:

docker ps

O PostgreSQL deve aparecer apenas como:

5432/tcp

E não como:

0.0.0.0:5432->5432/tcp

Isso confirma que o banco não está exposto para a máquina host.

9. Serviço Wiki.js

O serviço wiki utiliza a imagem:

image: requarks/wiki:2

A aplicação Wiki.js é exposta externamente pela porta 3000:

ports:
  - "3000:3000"

Com isso, a aplicação pode ser acessada no navegador pelo endereço:

http://localhost:3000

Apenas essa porta é exposta, mantendo o banco isolado.

10. Rede Interna Docker

Os dois serviços estão conectados à rede:

wiki-network

Essa rede permite que o Wiki.js se comunique com o PostgreSQL usando o nome do serviço db como host do banco.

No .env, isso é definido por:

DB_HOST=db

Dessa forma, a aplicação acessa o banco internamente, sem necessidade de usar IP fixo ou expor a porta do PostgreSQL para fora do Docker.

11. Persistência de Dados

Foram configurados volumes nomeados para garantir que os dados não sejam perdidos quando os containers forem removidos ou recriados.

Volume do PostgreSQL
postgres_data:/var/lib/postgresql/data

Esse volume armazena os dados reais do banco PostgreSQL.

Volume do Wiki.js
wiki_storage:/wiki/storage

Esse volume preserva arquivos, uploads e dados locais relacionados ao Wiki.js.

A lógica adotada é:

Container pode ser removido.
Volume permanece.
Dados continuam salvos.
12. Healthcheck e Ordem de Inicialização

O PostgreSQL possui um healthcheck configurado com pg_isready:

healthcheck:
  test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
  interval: 10s
  timeout: 5s
  retries: 5

Esse comando verifica se o banco está realmente pronto para receber conexões.

Isso é importante porque existe diferença entre:

Container ligado

e

Banco pronto para conexão

Sem o healthcheck, o Wiki.js poderia tentar iniciar antes do PostgreSQL estar disponível, causando falha na aplicação.

Para resolver isso, o serviço wiki foi configurado com:

depends_on:
  db:
    condition: service_healthy

Assim, o Wiki.js só inicia depois que o PostgreSQL estiver com status healthy.

13. Execução do Projeto
13.1 Clonar o repositório
git clone https://github.com/olicosta/docker_wiki.js.git
13.2 Entrar na pasta do projeto
cd docker_wiki.js
13.3 Criar o arquivo .env

Copie o conteúdo do .env.example para um novo arquivo chamado .env.

No Windows PowerShell:

copy .env.example .env

Depois edite o arquivo .env com os valores reais.

13.4 Subir os containers
docker compose up -d
13.5 Verificar os serviços
docker compose ps

Resultado esperado:

wikijs-db    Up    healthy
wikijs-app   Up
13.6 Acessar a aplicação

Abra no navegador:

http://localhost:3000
14. Teste de Persistência

Para validar a persistência dos dados, foi criada uma página dentro do Wiki.js chamada:

Projeto Final Docker

Depois, o ambiente foi derrubado com:

docker compose down

Esse comando remove os containers e a rede criada pelo Docker Compose, mas mantém os volumes nomeados.

Em seguida, o ambiente foi iniciado novamente:

docker compose up -d

Após a reinicialização, a página criada continuou disponível no Wiki.js.

Isso comprova que os dados não foram perdidos, pois estão armazenados nos volumes nomeados postgres_data e wiki_storage.

15. Validação Técnica
15.1 Verificação dos containers

Comando utilizado:

docker compose ps

Resultado observado:

NAME         IMAGE             SERVICE   STATUS
wikijs-app   requarks/wiki:2   wiki      Up
wikijs-db    postgres:16       db        Up (healthy)

O status healthy no container wikijs-db confirma que o healthcheck foi executado corretamente.

15.2 Verificação das portas

Comando utilizado:

docker ps

Resultado observado:

wikijs-app   0.0.0.0:3000->3000/tcp
wikijs-db    5432/tcp

Interpretação:

O Wiki.js está acessível externamente pela porta 3000;
O PostgreSQL não está exposto para a máquina host;
O banco aparece apenas como 5432/tcp, sem mapeamento externo;
A comunicação entre aplicação e banco ocorre somente pela rede interna Docker.
16. Boas Práticas Aplicadas
16.1 Zero Hardcoding

Nenhuma senha ou credencial foi escrita diretamente no docker-compose.yml.

As informações sensíveis são carregadas pelo arquivo .env.

16.2 Uso de .env.example

O arquivo .env.example foi incluído no repositório para servir como modelo de configuração.

Ele permite que outro usuário saiba quais variáveis precisa configurar para executar o projeto.

16.3 Proteção do .env

O arquivo .env real foi incluído no .gitignore.

Conteúdo do .gitignore:

.env

Isso evita o envio de credenciais reais ao GitHub.

16.4 Isolamento de Rede

Foi criada uma rede interna chamada wiki-network.

Os containers se comunicam por essa rede, sem expor o banco de dados para fora.

16.5 Persistência com Volumes Nomeados

Foram usados volumes nomeados para armazenar dados do banco e da aplicação.

Volumes configurados:

volumes:
  postgres_data:
  wiki_storage:
16.6 Healthcheck

O PostgreSQL utiliza pg_isready para validar se está pronto para receber conexões.

O Wiki.js depende desse status para iniciar corretamente.

17. Comandos Principais

Subir o ambiente:

docker compose up -d

Verificar containers do Compose:

docker compose ps

Verificar containers Docker:

docker ps

Derrubar o ambiente mantendo volumes:

docker compose down

Subir novamente:

docker compose up -d

Acessar a aplicação:

http://localhost:3000
18. Comando que NÃO deve ser usado no teste de persistência

Durante o teste de persistência, não deve ser utilizado:

docker compose down -v

O parâmetro -v remove os volumes nomeados.

Se esse comando for executado, os dados armazenados no PostgreSQL e no Wiki.js podem ser apagados.

O comando correto para testar persistência é:

docker compose down
19. Resultado Final

O projeto executa corretamente o Wiki.js com PostgreSQL utilizando Docker Compose.

A aplicação fica disponível em:

http://localhost:3000

O banco PostgreSQL permanece isolado, sem exposição externa da porta 5432.

Os dados persistem após reinicialização dos containers, graças ao uso dos volumes nomeados.

O Wiki.js só inicia após o PostgreSQL estar saudável, devido ao uso de healthcheck com pg_isready e depends_on com condition: service_healthy.

20. Conclusão

Este projeto demonstra a implantação de uma infraestrutura conteinerizada utilizando Docker Compose, com foco em segurança, persistência, isolamento e resiliência.

A solução atende aos principais requisitos técnicos propostos:

Arquitetura em duas camadas;
Wiki.js como camada de aplicação;
PostgreSQL como camada de dados;
Banco de dados sem exposição externa;
Aplicação acessível pela porta 3000;
Comunicação interna por rede Docker;
Persistência com volumes nomeados;
Credenciais fora do docker-compose.yml;
Uso de .env.example;
.env protegido pelo .gitignore;
Healthcheck com pg_isready;
Inicialização controlada da aplicação;
Teste prático de persistência validado.

Com essa configuração, o ambiente fica mais seguro, organizado e adequado para uma implantação baseada em containers.
