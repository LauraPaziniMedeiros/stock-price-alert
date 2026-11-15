# Stock Price Alert

> 🇺🇸 English | [🇧🇷 Versão em Português](#alerta-de-cotação)

Console application developed in C# (.NET 10) that continuously monitors the real-time market price of an asset traded on B3 and dispatches email recommendations when prices breach defined sell or buy thresholds.

---

## Key Features

- **Real-Time Market Monitoring**: Fetches live quotes at configurable intervals using the [Brapi API](https://brapi.dev).
- **Automated Email Alerts**: Dispatches advice emails via SMTP (supporting TLS/STARTTLS) when prices rise above sell targets or drop below buy targets.
- **Anti-Spam State Machine (Hysteresis)**: Prevents inbox flooding by triggering an email only upon crossing a threshold and re-arming once the price returns to normal range.
- **Layered Configuration**: Keeps default settings in `appsettings.json` while allowing secret overrides through `appsettings.local.json`.
- **Automated Test Suite**: 100% offline unit tests implemented with **xUnit** and **Moq**.
- **Containerization**: Production-ready, multi-stage Docker build running as a non-privileged user (`app`).

---

## Architecture

```text
stock-price-alert-api/
├── StockPriceAlert.slnx                 # Root solution linking app and tests
├── Dockerfile                          # Multi-stage build (restore -> test -> publish -> runtime)
├── .dockerignore                       # Excludes local caches, binaries, and secrets
├── StockPriceAlert/                    # Core Console Application
│   ├── Configuration/                  # Strongly-typed POCOs for appsettings
│   ├── Models/                         # Domain records (StockQuote, AlertThresholds)
│   ├── Services/                       # Service interfaces and implementations
│   │   ├── IStockService.cs / BrapiStockService.cs
│   │   ├── IEmailService.cs / EmailService.cs
│   │   └── StockMonitorEngine.cs       # Core evaluation and state logic
│   ├── appsettings.json                # Base configuration defaults
│   └── Program.cs                      # CLI parsing, bootstrapping, and execution loop
└── StockPriceAlert.Tests/              # xUnit Test Suite
    └── StockMonitorEngineTests.cs      # Unit tests for threshold boundaries & anti-spam
```

---

## Configuration

The application loads configuration in layers:
1. `appsettings.json`: Public defaults (base URLs, default ports).
2. `appsettings.local.json`: Local overrides containing your private credentials.

### Setup Instructions:
1. Create your local settings file from the template:
   ```bash
   cp StockPriceAlert/appsettings.example.json StockPriceAlert/appsettings.local.json
   ```
2. Configure your SMTP credentials in `StockPriceAlert/appsettings.local.json`:
   ```json
   {
     "EmailSettings": {
       "SmtpServer": "smtp.gmail.com",
       "Port": 587,
       "SenderName": "Stock Alert System",
       "SenderEmail": "your-email@gmail.com",
       "SenderPassword": "your-16-char-app-password",
       "RecipientEmail": "destination@example.com"
     },
     "StockApi": {
       "BaseUrl": "https://brapi.dev/api/quote/",
       "ApiKey": "",
       "IntervalSeconds": 60
     }
   }
   ```
   > **Note for Gmail users**: Google requires an **App Password** for third-party SMTP connections. Generate one under **Google Account -> Security -> 2-Step Verification -> App passwords**.

---

## How to Run

Run these commands from the repository root. Press Ctrl+C to stop the monitor.

### Method 1: Native .NET CLI

**Run the monitor:**
```bash
dotnet run --project StockPriceAlert -- <SYMBOL> <SELL_TARGET> <BUY_TARGET>

# Example:
dotnet run --project StockPriceAlert -- PETR4 22.67 22.59
```

**Run Automated Unit Tests:**
```bash
dotnet test
```

---

### Method 2: Docker Compose

The included `docker-compose.yml` builds the image and mounts the local settings file read-only. Create and fill in `StockPriceAlert/appsettings.local.json` as described in [Configuration](#configuration), then run:

**1. Build the Docker image:**
```bash

docker compose build
```
*(Unit tests run automatically during the build; the image build aborts if any test fails)*.

**2. Run with your local configuration file:**
```bash
docker compose run --rm stock-alert <SYMBOL> <SELL_TARGET> <BUY_TARGET>

# Example
docker compose run --rm stock-alert PETR4 22.67 22.59
```
---

## Next Steps

The project already covers the core MVP, and the next enhancements are focused on extensibility and scalability:

- **Add `StockServiceFactory` for provider switching**: keep the stock-fetching implementation pluggable through `appsettings.json`, so the app can switch providers without changing `Program.cs` or recompiling the project.
- **Support multi-currency and global exchanges**: extend `StockQuote` and the monitoring logic to handle values such as USD/EUR and listings from international exchanges while formatting prices appropriately for the active market.
- **Add a Yahoo Finance provider**: implement a second stock service provider and register it in the factory to demonstrate true provider abstraction and easier future integrations.

---

## AI use

This project used AI as a practical implementation accelerator for architecture, documentation and validation work:

- AI-assisted planning helped refine the initial C# project layout and and Docker-based production approach.
- AI was used for documentation, such as the README and project notes.
- AI helped elaborate unit testing with `StockPriceAlert.Tests`.
- AI supported code review and refinement by improving clarity, consistency, and early validation of implementation decisions.

---

# Alerta de Cotação

> [🇺🇸 English](#stock-price-alert) | 🇧🇷 Versão em Português

Aplicação de console desenvolvida em C# (.NET 10) que monitora continuamente a cotação em tempo real de ativos negociados na B3 e envia recomendações por e-mail quando os preços ultrapassam os limites definidos para venda ou compra.

---

## Principais funcionalidades

- **Monitoramento do mercado em tempo real**: consulta cotações atuais em intervalos configuráveis usando a [API Brapi](https://brapi.dev).
- **Alertas automáticos por e-mail**: envia recomendações via SMTP (com suporte a TLS/STARTTLS) quando os preços superam os limites de venda ou caem abaixo dos limites de compra.
- **Máquina de estados anti-spam (histerese)**: evita excesso de e-mails, enviando um alerta apenas ao cruzar um limite e rearmando o alerta quando o preço retorna à faixa normal.
- **Configuração em camadas**: mantém as configurações padrão em `appsettings.json` e permite substituí-las localmente por configurações privadas em `appsettings.local.json`.
- **Suíte de testes automatizados**: testes unitários 100% offline implementados com **xUnit** e **Moq**.
- **Conteinerização**: build Docker multiestágio pronto para produção, executado com um usuário sem privilégios (`app`).

---

## Arquitetura

```text
stock-price-alert-api/
├── StockPriceAlert.slnx                 # Solução raiz que reúne a aplicação e os testes
├── Dockerfile                          # Build multiestágio (restauração -> testes -> publicação -> execução)
├── .dockerignore                       # Exclui caches locais, binários e segredos
├── StockPriceAlert/                    # Aplicação de console principal
│   ├── Configuration/                  # Classes POCO fortemente tipadas para o appsettings
│   ├── Models/                         # Modelos de domínio (StockQuote, AlertThresholds)
│   ├── Services/                       # Interfaces e implementações dos serviços
│   │   ├── IStockService.cs / BrapiStockService.cs
│   │   ├── IEmailService.cs / EmailService.cs
│   │   └── StockMonitorEngine.cs       # Lógica principal de avaliação e estado
│   ├── appsettings.json                # Configurações padrão
│   └── Program.cs                      # Leitura de argumentos CLI, inicialização e loop de execução
└── StockPriceAlert.Tests/              # Suíte de testes xUnit
    └── StockMonitorEngineTests.cs      # Testes unitários dos limites e da prevenção contra alertas repetidos
```

---

## Configuração

O aplicativo carrega as configurações em camadas:
1. `appsettings.json`: configurações padrão públicas (URLs base, portas padrão).
2. `appsettings.local.json`: configurações locais que substituem as padrão e contêm suas credenciais privadas.

### Instruções de configuração:
1. Crie seu arquivo de configurações locais a partir do modelo:
   ```bash
   cp StockPriceAlert/appsettings.example.json StockPriceAlert/appsettings.local.json
   ```
2. Configure suas credenciais SMTP em `StockPriceAlert/appsettings.local.json`:
   ```json
   {
     "EmailSettings": {
       "SmtpServer": "smtp.gmail.com",
       "Port": 587,
       "SenderName": "Sistema de Alertas de Ações",
       "SenderEmail": "seu-email@gmail.com",
       "SenderPassword": "sua-senha-de-app-de-16-caracteres",
       "RecipientEmail": "destino@example.com"
     },
     "StockApi": {
       "BaseUrl": "https://brapi.dev/api/quote/",
       "ApiKey": "",
       "IntervalSeconds": 60
     }
   }
   ```
   > **Observação para usuários do Gmail**: o Google exige uma **senha de app** para conexões SMTP de terceiros. Gere uma em **Conta do Google -> Segurança -> Verificação em duas etapas -> Senhas de app**.

---

## Como executar

Execute estes comandos na raiz do repositório. Pressione Ctrl+C para parar o monitor.

### Método 1: CLI nativa do .NET

**Executar o monitor:**
```bash
dotnet run --project StockPriceAlert -- <ATIVO> <PRECO_VENDA> <PRECO_COMPRA>

# Exemplo:
dotnet run --project StockPriceAlert -- PETR4 22.67 22.59
```

**Executar os testes unitários automatizados:**
```bash
dotnet test
```

---

### Método 2: Docker Compose

O arquivo `docker-compose.yml` incluído constrói a imagem e monta o arquivo de configurações locais em modo somente leitura. Crie e preencha `StockPriceAlert/appsettings.local.json` conforme descrito em [Configuração](#configuração) e execute:

**1. Construir a imagem Docker:**
```bash

docker compose build
```
*(Os testes unitários são executados automaticamente durante o build; se algum teste falhar, o build da imagem é interrompido.)*

**2. Executar com seu arquivo de configurações locais:**
```bash
docker compose run --rm stock-alert <ATIVO> <PRECO_VENDA> <PRECO_COMPRA>

# Exemplo
docker compose run --rm stock-alert PETR4 22.67 22.59
```

---

## Próximos Passos

O projeto já cobre o MVP principal, e as próximas melhorias têm como foco a extensibilidade e a escalabilidade:

- **Adicionar um provedor do Yahoo Finance**: implementar um segundo provedor de serviços de ações e registrá-lo na factory para demonstrar uma abstração real de provedores e facilitar integrações futuras.

- **Adicionar `StockServiceFactory` para alternância de provedores**: manter a implementação de busca de ações plugável por meio do appsettings.json, permitindo que a aplicação alterne entre provedores sem precisar alterar o Program.cs ou recompilar o projeto.

- **Suportar múltiplas moedas e bolsas globais**: estender `StockQuote` e a lógica de monitoramento para lidar com valores como USD/EUR e listagens de bolsas internacionais, formatando os preços adequadamente para o mercado ativo.

## Uso de IA

Este projeto utilizou IA como um acelerador prático de implementação para arquitetura, documentação e validação:

- O planejamento assistido por IA ajudou a refinar a estrutura inicial do projeto em C# e a abordagem de produção baseada em Docker.

- A IA foi utilizada para a documentação, como o README e as notas do projeto.

- A IA auxiliou na elaboração de testes unitários com o `StockPriceAlert.Tests`.

- A IA apoiou a revisão e o refinamento de código, melhorando a clareza, a consistência e a validação antecipada das decisões de implementação.