# ⚡ BarberPoint - Azure Function (Relatórios)

## Descrição

Azure Function HTTP Trigger que gera e calcula relatórios de agendamentos, respondendo requisições de agregação de dados. Faz parte de uma aplicação distribuída que combina **microservices**, **BFF**, **microfrontend** e **serverless**.

## Arquitetura

- `GenerateScheduleReport/index.js` — lógica principal do HTTP Trigger
- `GenerateScheduleReport/function.json` — binding HTTP e rota da função
- `host.json` — configuração do runtime do Azure Functions
- `package.json` — dependências e scripts

Padrões aplicados:
- **Serverless** com Azure Functions
- **API Gateway + BFF + Microservice + Database Service + Serverless**
- **Event-Driven Architecture** para relatórios e métricas em fluxo distribuído

## Responsabilidades

- Calcular estatísticas de agendamentos
- Enriquecer dados com métricas
- Retornar relatório consolidado

## Resposta

```json
{
  "totalClientes": 50,
  "totalReceita": 2500.00,
  "taxaOcupacao": 85,
  "tempoMedioAtendimento": 45,
  "agendamentosRealizado": 120,
  "agendamentosCancelados": 5,
  "dataGeracao": "2026-04-27T14:30:00Z"
}
```

## Tecnologias

- **Node.js 20**
- **Azure Functions**
- **Express.js** (local testing)
- **Jest** (testes)
- **Axios** (HTTP calls)

## Pré-requisitos

- Node.js 18+
- npm
- Azure Functions Core Tools (opcional)
- Azure CLI (para deploy)

## Como Rodar Localmente

### 1. Instalação

```bash
git clone https://github.com/seu-usuario/barberpoint-azure-function.git
cd barberpoint-azure-function

npm install
```

### 2. Desenvolvimento Local

```bash
# Com Azure Functions Core Tools
func start

# Função estará disponível em:
# http://localhost:7071/api/reports/schedule-report
```

### 3. Teste Local com cURL

```bash
curl -X POST http://localhost:7071/api/reports/schedule-report \
  -H "Content-Type: application/json" \
  -d '{"agendamentoId": "123"}'
```

## Estrutura

```
GenerateScheduleReport/
├── index.js           # Lógica da função
├── function.json      # Configuração
└── package.json
```

## Código Exemplo (index.js)

```javascript
module.exports = async function (context, req) {
  context.log('HTTP trigger function processed a request.');

  try {
    // Lógica de cálculo
    const relatorio = {
      totalClientes: 50,
      totalReceita: 2500.00,
      taxaOcupacao: 85,
      tempoMedioAtendimento: 45,
      dataGeracao: new Date().toISOString()
    };

    context.res = {
      status: 200,
      body: relatorio
    };
  } catch (error) {
    context.res = {
      status: 500,
      body: { error: error.message }
    };
  }
};
```

## Testes

```bash
# Executar testes
npm test

# Com cobertura
npm test -- --coverage
```

## Deploy no Azure

### 1. Autenticação

```bash
az login
```

### 2. Criar Resource Group

```bash
az group create --name barberpoint-rg --location eastus
```

### 3. Criar Storage Account

```bash
az storage account create \
  --name barberpointstorage \
  --resource-group barberpoint-rg \
  --location eastus \
  --sku Standard_LRS
```

### 4. Criar Function App

```bash
az functionapp create \
  --resource-group barberpoint-rg \
  --consumption-plan-location eastus \
  --runtime node \
  --runtime-version 20 \
  --functions-version 4 \
  --name barberpoint-function \
  --storage-account barberpointstorage
```

### 5. Deploy

```bash
func azure functionapp publish barberpoint-function
```

## Variáveis de Ambiente

```bash
# .env (local)
AZURE_SUBSCRIPTION_ID=xxx
AZURE_TENANT_ID=xxx
AZURE_CLIENT_ID=xxx
AZURE_CLIENT_SECRET=xxx
```

## Endpoint de Produção

```
POST https://barberpoint-function.azurewebsites.net/api/reports/schedule-report
```

## Monitoramento

```bash
# Ver logs
az functionapp log tail \
  --resource-group barberpoint-rg \
  --name barberpoint-function

# Ver métricas
az monitor metrics list \
  --resource barberpoint-function \
  --resource-group barberpoint-rg
```

## CI/CD com GitHub Actions

### .github/workflows/deploy.yml

```yaml
name: Deploy Function

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm install
      
      - name: Run tests
        run: npm test
      
      - name: Deploy to Azure
        uses: Azure/functions-action@v1
        with:
          app-name: 'barberpoint-function'
          package: '.'
          publish-profile: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE }}
```

---

## Autores
- Irvanlei de Abreu
- João Yutaka
- Fellipe
- Nelson
- Allan