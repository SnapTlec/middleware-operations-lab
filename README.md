# LAB-03 — Sustentação Middleware Enterprise

![Story](https://img.shields.io/badge/type-Story-0052CC?style=flat-square)
![High Priority](https://img.shields.io/badge/priority-High-E5493A?style=flat-square)
![Middleware](https://img.shields.io/badge/epic-Middleware-6B48C7?style=flat-square)
![Status](https://img.shields.io/badge/status-Operational-2DA44E?style=flat-square)
![Sprint](https://img.shields.io/badge/sprint-Sprint%202-0052CC?style=flat-square)
![Estimate](https://img.shields.io/badge/estimate-3--4%20dias-555555?style=flat-square)

---

## Identificação

| Campo       | Valor                        |
|-------------|------------------------------|
| ID          | `INFRA-003`                  |
| Epic        | Portfólio Enterprise         |
| Tipo        | Story                        |
| Prioridade  | High                         |
| Sprint      | Sprint 2                     |
| Estimativa  | 3–4 dias                     |
| Assignee    | @seu-usuario                 |
| Repositório | `middleware-operations-lab`       |
| Status      | Operational                  |

---

## Objetivo

Criar um handbook operacional de referência para sustentação de ambientes WildFly em produção, cobrindo o ciclo completo de operação: instalação, configuração de datasources, deploy de aplicações Java EE, análise de logs, troubleshooting de falhas comuns e tuning básico de JVM e connection pool. O repositório deve funcionar como guia real de sustentação que um sysadmin consultaria durante um incidente.

---

## Descrição

O lab é estruturado em quatro seções principais.

**Provisionamento**
Instalação do WildFly em modo standalone, configuração do `standalone.xml`, abertura de portas de management (9990 HTTP / 9993 HTTPS), criação de usuário admin via `add-user.sh` e acesso ao console web.

**Datasources**
Instalação do driver JDBC (Oracle e SQL Server) como módulo do WildFly, criação de datasource via `jboss-cli.sh` com parâmetros corretos de connection pool (`min-pool-size`, `max-pool-size`, `idle-timeout-minutes`), validação com `test-connection-in-pool` e exemplos de `datasource.xml` para ambos os bancos.

**Deploy**
Ciclo completo: deploy via console web, via CLI (`deploy --force`), via diretório `deployments/`, verificação de status (`deployment-info`), undeploy e rollback. Inclui tabela de marcadores de deploy com seu significado operacional.

**Troubleshooting Handbook**
12 cenários reais de falha, cada um com sintoma observado, localização do log relevante, comando de diagnóstico, causa raiz mais comum e solução aplicada.

---

## Arquitetura do ambiente de lab

```
┌─────────────────────────────────────────┐
│           Docker Compose Network        │
│                                         │
│  ┌──────────────────────────────────┐   │
│  │           WildFly 30             │   │
│  │  porta: 8080 (app)               │   │
│  │  porta: 9990 (management)        │   │
│  │                                  │   │
│  │  App: enterprise-app.war ──────► │   │
│  │  Datasource: AppDS (SQL Server)  │   │
│  │  Datasource: OracleDS            │   │
│  └──────────────┬───────────────────┘   │
│                 │ JDBC                  │
│     ┌───────────┴─────────────┐         │
│     ▼                         ▼         │
│  ┌─────────┐           ┌──────────┐     │
│  │SQL Srv  │           │Oracle XE │     │
│  │  :1433  │           │  :1521   │     │
│  └─────────┘           └──────────┘     │
└─────────────────────────────────────────┘
```

---

## Stack técnica

![WildFly](https://img.shields.io/badge/WildFly-30-EE0000?style=flat-square)
![Oracle](https://img.shields.io/badge/Oracle-XE%2021c-F80000?style=flat-square&logo=oracle&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL%20Server-2022-CC2927?style=flat-square&logo=microsoftsqlserver&logoColor=white)
![Java EE](https://img.shields.io/badge/Java%20EE-WAR%20Deploy-007396?style=flat-square&logo=java&logoColor=white)
![Docker](https://img.shields.io/badge/Docker%20Compose-v3.9-2496ED?style=flat-square&logo=docker&logoColor=white)

---

## Pré-requisitos

- Docker Engine 24+ e Docker Compose v2
- Mínimo 6 GB de RAM (WildFly: 1GB, SQL Server: 2GB, Oracle XE: 2GB)
- Portas livres: `8080`, `9990`, `1433`, `1521`
- Driver JDBC: `mssql-jdbc-12.x.jar` e `ojdbc11.jar` (baixar manualmente — licença não permite redistribuição)

---

## Como executar

```bash
git clone https://github.com/SnapTlec/middleware-operations-lab.git
cd middleware-operations-lab

# Coloque os drivers JDBC em drivers/
ls drivers/
# mssql-jdbc-12.x.jar
# ojdbc11.jar

# Suba o ambiente
docker-compose up -d

# Provisione datasources (aguarda WildFly estar UP)
./scripts/provision-sqlserver.sh
./scripts/provision-oracle.sh

# Deploy da aplicação de teste
./scripts/deploy-app.sh app/enterprise-app.war

# Valide end-to-end
curl http://localhost:8080/enterprise-app/api/status
```

---

## Estrutura do repositório

```
middleware-operations-lab/
├── wildfly/
│   ├── Dockerfile
│   └── modules/          ← módulos JDBC instalados aqui
├── sqlserver/
│   └── init.sql
├── oracle/
│   └── init.sql
├── app/
│   └── enterprise-app.war
├── scripts/
│   ├── provision-sqlserver.sh
│   ├── provision-oracle.sh
│   └── deploy-app.sh
├── docs/
│   ├── troubleshooting-handbook.md   ← 12 cenários de falha
│   ├── tuning-guide.md               ← JVM e connection pool
│   └── deploy-guide.md               ← ciclo completo de deploy
├── docker-compose.yml
└── README.md
```

---

## Marcadores de deploy WildFly

| Arquivo            | Significado                                      |
|--------------------|--------------------------------------------------|
| `app.war`          | WAR presente no diretório de deploy              |
| `app.war.isdeploying` | Deploy em andamento                           |
| `app.war.deployed` | Deploy concluído com sucesso                     |
| `app.war.failed`   | Deploy falhou — verificar `server.log`           |
| `app.war.dodeploy` | Sinaliza re-deploy sem resubir o WildFly         |
| `app.war.undeployed` | Aplicação removida com sucesso                 |

---

## Tarefas de implementação

- [ ] Container WildFly 30 + SQL Server + Oracle XE via Docker Compose
- [ ] Script `provision-sqlserver.sh`: instalar driver JDBC via CLI, criar datasource com pool e validar conexão
- [ ] Script `provision-oracle.sh`: mesma sequência para Oracle com driver OJDBC
- [ ] Aplicação WAR de exemplo com servlet que lê do banco e retorna JSON
- [ ] Script `deploy-app.sh` com verificação de status e timeout configurável
- [ ] Documento `troubleshooting-handbook.md` com os 12 cenários de falha detalhados
- [ ] Documento `tuning-guide.md` com parâmetros JVM e connection pool por porte de ambiente
- [ ] Capturas de tela: console management, datasource configurado, test-connection, deploy bem-sucedido
- [ ] README com fluxo operacional completo e índice do handbook

---

## Critérios de aceite

- [ ] `test-connection-in-pool` retorna `true` para SQL Server e Oracle
- [ ] Aplicação WAR implantada com sucesso, lê do banco e responde HTTP 200
- [ ] Handbook cobre 12 cenários com sintoma, diagnóstico e solução
- [ ] Tuning guide especifica parâmetros com valores de referência por porte (pequeno / médio / grande)
- [ ] Toda a seção de deploy documentada com comandos CLI reais executáveis

---

## Troubleshooting Handbook — índice dos 12 cenários

| # | Cenário                                      | Nível    |
|---|----------------------------------------------|----------|
| 1 | Datasource connection refused                | Critical |
| 2 | ClassNotFoundException do driver JDBC        | High     |
| 3 | OutOfMemoryError na JVM                      | Critical |
| 4 | Deployment timeout                           | High     |
| 5 | Porta de management inacessível              | Medium   |
| 6 | Aplicação retornando 503                     | High     |
| 7 | Connection pool esgotado                     | Critical |
| 8 | Erro de SSL no datasource                    | Medium   |
| 9 | Problema de permissão em arquivo de log      | Low      |
| 10 | Falha de deploy por dependência ausente     | High     |
| 11 | Timeout de transaction                       | High     |
| 12 | Lock de banco bloqueando aplicação           | Critical |

> Detalhes completos em [`docs/troubleshooting-handbook.md`](docs/troubleshooting-handbook.md)

---

## Tuning de referência por porte

| Parâmetro            | Pequeno (≤2GB) | Médio (4–8GB) | Grande (16GB+) |
|----------------------|----------------|---------------|----------------|
| `-Xms`               | 256m           | 1g            | 4g             |
| `-Xmx`               | 512m           | 2g            | 8g             |
| `-XX:MetaspaceSize`  | 128m           | 256m          | 512m           |
| `min-pool-size`      | 5              | 10            | 20             |
| `max-pool-size`      | 20             | 50            | 100            |
| `idle-timeout`       | 5 min          | 10 min        | 15 min         |

---

## Referências

- [WildFly Admin Guide — Datasources](https://docs.wildfly.org/30/Admin_Guide.html#DataSource)
- [WildFly CLI Guide](https://docs.wildfly.org/30/Admin_Guide.html#CLI_Recipes)
- [mssql-jdbc GitHub](https://github.com/microsoft/mssql-jdbc)
- [Oracle JDBC Downloads](https://www.oracle.com/database/technologies/appdev/jdbc-downloads.html)
