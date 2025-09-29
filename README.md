# Deploy do Projeto na Azure

### 1. Setup Inicial do Projeto (Git Clone)
⚠️OBS: O código fonte está no repositório abaixo↓

Para começar, clone o projeto e navegue para o branch **Nicolas**:

```bash
# 1. Clonar o repositório
git clone https://github.com/jjosebastos/mottu-challenge.git

# 2. Navegar para o diretório
cd mottu-challenge

# 3. Mudar para o branch Nicolas
git checkout Nicolas
```
### 2. Build do Projeto
Foi gerado o arquivo .jar utilizando o Maven:
```bash
mvn clean package -DskipTests
```

### 3. Build e Push da Imagem Docker
A aplicação foi empacotada em uma imagem Docker e enviada para o Azure Container Registry (ACR):
```bash
# Build da imagem
docker build -t mottuchallengeacr-atgycfb5fggvhtg2.azurecr.io/task-manager:v1 .

# Push para o ACR
docker push mottuchallengeacr-atgycfb5fggvhtg2.azurecr.io/task-manager:v1
```
OBS: O Dockerfile está no repositório https://github.com/jjosebastos/mottu-challenge.git na branch Nicolas

## 4. CRIAÇÃO E CONFIGURAÇÃO DA INFRAESTRUTURA BASE NO AZURE
Este resumo detalha o processo de criação dos serviços essenciais para a hospedagem e persistência da aplicação no Azure.

### 1. Configuração do Azure SQL Server e Database

O banco de dados foi configurado em duas etapas:

#### 1.1. Criação do SQL Server e Database
- Um **Azure SQL Server** (`mottu-server`) foi criado para hospedar o banco de dados.  
- Um **Azure SQL Database** (`mottu-db`) foi provisionado dentro deste servidor.  
- A segurança foi configurada via **Firewall**, garantindo que a regra *"Allow Azure services and resources to access this server"* estivesse ativada.  
  Isso permitiu que o serviço de aplicação (**ACI/App Service**) pudesse se conectar ao banco de dados.

#### 1.2. Execução dos `scripts_db.sql`
- Após a criação, o arquivo **scripts_db.sql** foi executado no **Query Editor** do Azure SQL.  
- Este script criou todas as tabelas e schemas necessários (`t_mtu_filial`, `t_mtu_user`, etc.), preparando o banco de dados para a persistência dos dados da API.

---

### 2. Criação do Azure Container Registry (ACR)

O serviço de repositório de imagens foi configurado para armazenar o Docker da aplicação:

- Foi criado um **Azure Container Registry** (`mottuchallengeacr`).  
- Para permitir que ferramentas locais (como o `docker push`) e serviços do Azure acessassem o repositório, o **Admin user** foi ativado em **Access Keys** do ACR.

---

### 3. Criação do Azure Container Instances (ACI)

O primeiro serviço de hospedagem para o contêiner foi o ACI:

- Uma instância do **Azure Container Instances** foi criada.  
- Esta instância foi configurada para puxar a imagem Docker do ACR (`mottuchallengeacr`).  
- A aba de **Networking** foi configurada para expor a porta **8080** (porta padrão do Spring Boot) publicamente.

## 7. Exemplos de Requests e Teste

```http
POST http://4.228.145.123:8080/users
Content-Type: application/json

{
    "email": "admin@mottu.com",
    "password": "mottu123",
    "role": "ADMIN"
}
```
```http
Resposta Esperada (201 Created):

json
{
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "email": "admin@mottu.com",
    "role": "ADMIN",
    "createdAt": "2024-01-15T10:30:00Z"
}
```
```http
POST http://4.228.145.123:8080/login
Content-Type: application/json

{
    "email": "admin@mottu.com",
    "password": "mottu123"
}
```
```http
Resposta Esperada (200 OK):

json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "email": "admin@mottu.com"
}
```
```http
POST http://4.228.145.123:8080/filial
Authorization: Bearer <TOKEN>
Content-Type: application/json

{
    "filialRequests": [
        {
            "nome": "Filial São Paulo",
            "cnpj": "12345678912345678", 
            "cdPais": "BR",
            "dataAbertura": "2023-01-15"
        }
    ]
}
```

```http
Resposta Esperada (201 Created):

json
{
    "filiais": [
        {
            "id": "1aa8355a-12bf-4ba4-bc30-7f649e29a958",
            "nome": "Filial São Paulo",
            "cnpj": "12345678912345678",
            "cdPais": "BR",
            "dataAbertura": "2023-01-15",
            "createdAt": "2024-01-15T10:35:00Z"
        }
    ]
}
```
```http
GET http://4.228.145.123:8080/filial/all
Authorization: Bearer <TOKEN>
```
```http
{
    "filiais": [
        {
            "idFilial": "1aa8355a-12bf-4ba4-bc30-7f649e29a958",
            "cnpj": "12345678912345678",
            "nome": "Filial São Paulo",
            "cdPais": "BR",
            "dataAbertura": "2023-01-15"
        }
    ],
}
```

```http

GET http://4.228.145.123:8080/filial/1aa8355a-12bf-4ba4-bc30-7f649e29a958
Authorization: Bearer <TOKEN>
```

```http
{
    "idFilial": "1aa8355a-12bf-4ba4-bc30-7f649e29a958",
    "cnpj": "12345678912345678",
    "nome": "Filial São Paulo",
    "cdPais": "BR",
    "dataAbertura": "2023-01-15"
}
```

```http
PUT http://4.228.145.123:8080/filial/1aa8355a-12bf-4ba4-bc30-7f649e29a958
Authorization: Bearer <TOKEN>
Content-Type: application/json

{
    "nome": "Filial São Paulo - Atualizada",
    "cnpj": "12345678912345678",
    "cdPais": "BR",
    "dataAbertura": "2023-01-15"
}
```
```http
{
    "idFilial": "1aa8355a-12bf-4ba4-bc30-7f649e29a958",
    "cnpj": "12345678912345678",
    "nome": "Filial São Paulo - Atualizada",
    "cdPais": "BR",
    "dataAbertura": "2023-01-15"
}
```
```http

DELETE http://4.228.145.123:8080/filial/1aa8355a-12bf-4ba4-bc30-7f649e29a958
Authorization: Bearer <TOKEN>
```
```http
{
    "message": "Filial deletada com sucesso"
}
```
