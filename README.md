# Docker Infra Prod – n8n, GLPI, Grafana & Komodo (GitOps via Komodo)
> **Status**: n8n em produção, gerenciado pelo Komodo. Os composes de GLPI, Grafana e Komodo estão no Git e entram pela mesma esteira conforme passam por validação. Bancos de dados relacionais (Postgres/MySQL) rodam fora do Docker, em Bare Metal, para performance e estabilidade.

Este repositório é a fonte da verdade da infra: os composes de cada serviço ficam versionados aqui, e o Komodo (IDP) sincroniza o estado do host com o Git via Resource Sync. Nada muda em produção sem um commit.
---
## Serviços principais
| Serviço | Descrição | Configuração | Dados/Volumes |
|---------|-----------|--------------|---------------|
| **n8n** | Automação com Puppeteer e OCI CLI | `opt/docker/n8n/` | `data/services/n8n/` |
| **GLPI** | Gestão de ativos e chamados | `opt/docker/glpi/` | `data/services/glpi/` |
| **Grafana** | Observabilidade e métricas | `opt/docker/grafana/` | `data/services/grafana/` |
| **Komodo** | Core & Periphery (Orquestrador) | `opt/docker/komo.do/` | `data/services/komodo/` |
---
## Arquitetura geral
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
## Configuração do n8n (imagem custom)
O n8n utiliza uma imagem personalizada para suportar automações complexas:
- **Chromium/Puppeteer**: Pré-configurado para web scraping.
- **OCI CLI**: Instalado em venv (`/opt/oci`) para automação na Oracle Cloud.
- **Node.js 22**: Runtime atualizado.
### Scripts
Localizados em `data/services/n8n/scripts/`. Exemplo: `scraper_tsplus.js`.
---
## Bancos de dados em Bare Metal
Diferente do n8n, GLPI e Grafana que rodam em Docker, seus bancos de dados estão fora do Docker para facilitar manutenção e performance:
- **n8n**: Conecta ao Postgres Bare Metal via `POSTGRES_HOST`.
- **GLPI & Grafana**: Conectam ao MySQL Bare Metal via `*_DB_HOST`.
- **Extra Hosts**: Os containers utilizam `host.docker.internal` para acessar o host quando necessário.
---
## Como subir os serviços

**Via Komodo (padrão):** os recursos declarados em [`komodo/`](./komodo) fazem o sync automaticamente — push no Git vira diff no Komodo, e o deploy é confirmado no IDP (ou automaticamente, via webhook). Veja [`komodo/README.md`](./komodo/README.md) para o primeiro import e os TODOs de servidor/stacks.

**Via Docker Compose (direto, fallback):**
```bash
# Exemplo: Subindo n8n
cd n8n
docker compose up -d
# Exemplo: Subindo GLPI
cd glpi
docker compose up -d
```
### Variáveis de Ambiente (.env)
Cada pasta em `opt/docker/` deve conter seu próprio `.env` ou um arquivo centralizado conforme a necessidade. Variáveis cruciais:
- IP/Host do Banco de Dados
- Credenciais de acesso
- Timezone (padrão `America/Sao_Paulo`)
---
## Estrutura de pastas
```
LAB-DEVOPS/
├── komodo/                 # Recursos declarados (Resource Sync do Komodo)
│   ├── README.md           # Como importar e operar o sync
│   ├── resource-sync.toml  # O sync que carrega os TOMLs abaixo
│   └── resources/
│       ├── server.toml     # O host de produção via Periphery
│       └── stacks.toml     # As stacks: n8n, GLPI, Grafana, Komodo
├── data/services/          # Volumes persistentes e scripts
│   ├── glpi/
│   ├── grafana/
│   ├── komodo/
│   └── n8n/
│       ├── n8n_data/
│       └── scripts/
├── glpi/                   # Compose da aplicação
├── grafana/
├── komo.do/                # Core, Mongo e Periphery
└── n8n/
    ├── Dockerfile          # Imagem custom (Chromium/Puppeteer + OCI CLI)
    └── compose.yaml
```
---
## Scripts do n8n
### scraper_tsplus.js
Script Puppeteer para extração de dados do TSPlus.
- **Uso**: Chamado via node dentro de Function Nodes no n8n.
- **Path no Container**: `/home/node/scripts/scraper_tsplus.js`.
---
## Manutenção
- **Logs**: `docker compose logs -f [service_name]`
- **Update**: `docker compose pull && docker compose up -d`
- **n8n Build**: `docker compose build --no-cache n8n`