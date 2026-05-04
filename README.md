# Docker Infra Prod – n8n, GLPI, Grafana & Komodo
> 🚀 **Status**: Somente o **n8n** está ativo em produção. Os demais serviços estão em fase de implementação.

Este repositório contém a configuração Docker para serviços críticos. Atualmente, o **n8n** (automação) é o serviço principal em operação, com **GLPI**, **Grafana** e **Komodo** em processo de configuração. Bancos de dados relacionais (Postgres/MySQL) são executados em **Bare Metal** para performance e estabilidade.
---
## ✨ Serviços Principais
| Serviço | Descrição | Configuração | Dados/Volumes |
|---------|-----------|--------------|---------------|
| **n8n** | Automação com Puppeteer e OCI CLI | `opt/docker/n8n/` | `data/services/n8n/` |
| **GLPI** | Gestão de ativos e chamados | `opt/docker/glpi/` | `data/services/glpi/` |
| **Grafana** | Observabilidade e métricas | `opt/docker/grafana/` | `data/services/grafana/` |
| **Komodo** | Core & Periphery (Orquestrador) | `opt/docker/komo.do/` | `data/services/komodo/` |
---
## 🧩 Arquitetura Geral
```text
[Servidor Host]
   ├─ Bare Metal:
   │    ├─ Postgres (n8n)
   │    └─ MySQL (GLPI, Grafana)
   │
   ├─ Docker (opt/docker/):
   │    ├─ n8n (Custom image c/ Puppeteer)
   │    ├─ GLPI
   │    ├─ Grafana
   │    └─ Komodo (Core, Mongo, Periphery)
   │
   └─ Volumes (data/services/):
        ├─ n8n_data, scripts/
        ├─ var_glpi
        ├─ var_grafana
        └─ komodo_data
```
---
## 🏗️ Configuração do n8n (Customizado)
O n8n utiliza uma imagem personalizada para suportar automações complexas:
- **Chromium/Puppeteer**: Pré-configurado para web scraping.
- **OCI CLI**: Instalado em venv (`/opt/oci`) para automação na Oracle Cloud.
- **Node.js 22**: Runtime atualizado.
### Scripts
Localizados em `data/services/n8n/scripts/`. Exemplo: `scraper_tsplus.js`.
---
## 📌 Bancos de Dados (Bare Metal)
Diferente do n8n, GLPI e Grafana que rodam em Docker, seus bancos de dados estão fora do Docker para facilitar manutenção e performance:
- **n8n**: Conecta ao Postgres Bare Metal via `POSTGRES_HOST`.
- **GLPI & Grafana**: Conectam ao MySQL Bare Metal via `*_DB_HOST`.
- **Extra Hosts**: Os containers utilizam `host.docker.internal` para acessar o host quando necessário.
---
## 🚀 Como subir os serviços
Para cada serviço, navegue até sua pasta em `opt/docker/` e use o docker-compose:
```bash
# Exemplo: Subindo n8n
cd opt/docker/n8n
docker compose up -d
# Exemplo: Subindo GLPI
cd opt/docker/glpi
docker compose up -d
```
### Variáveis de Ambiente (.env)
Cada pasta em `opt/docker/` deve conter seu próprio `.env` ou um arquivo centralizado conforme a necessidade. Variáveis cruciais:
- IP/Host do Banco de Dados
- Credenciais de acesso
- Timezone (padrão `America/Sao_Paulo`)
---
## 📂 Estrutura de Pastas Atualizada
```
Docker-Infra-Prod/
├── data/services/            # Volumes persistentes e scripts
│   ├── glpi/
│   ├── grafana/
│   └── n8n/
│       ├── n8n_data/
│       └── scripts/
└── opt/docker/               # Configurações de orquestração
    ├── glpi/
    ├── grafana/
    ├── komo.do/
    └── n8n/
        ├── Dockerfile
        └── docker-compose.yml
```
---
## 📝 Scripts n8n
### scraper_tsplus.js
Script Puppeteer para extração de dados do TSPlus.
- **Uso**: Chamado via node dentro de Function Nodes no n8n.
- **Path no Container**: `/home/node/scripts/scraper_tsplus.js`.
---
## 🛠️ Manutenção
- **Logs**: `docker compose logs -f [service_name]`
- **Update**: `docker compose pull && docker compose up -d`
- **n8n Build**: `docker compose build --no-cache n8n`