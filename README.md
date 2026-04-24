# CI/CD com GitHub Actions

Repositorio de pipeline CI/CD para validacao de qualidade da EBAC Shop com GitHub Actions.

## Sumario

- [Visao Geral](#visao-geral)
- [Stack e Versoes](#stack-e-versoes)
- [Arquitetura da Pipeline](#arquitetura-da-pipeline)
- [Cenarios Cobertos](#cenarios-cobertos)
- [Pre-requisitos](#pre-requisitos)
- [Instalacao](#instalacao)
- [Execucao da Pipeline](#execucao-da-pipeline)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Explicacao do Teste neste Repo Isolado](#explicacao-do-teste-neste-repo-isolado)
- [Troubleshooting](#troubleshooting)
- [Boas Praticas para Evolucao](#boas-praticas-para-evolucao)
- [Autor](#autor)
- [Licenca](#licenca)

## Visao Geral

Este projeto centraliza a pipeline de qualidade para execucao automatizada de testes por camada:

- API (Jest + Supertest)
- UI (Cypress)
- Performance (k6)

Workflow principal:

- `.github/workflows/qa-pipeline.yml`

Espelho documental:

- `github-actions/qa-pipeline.yml`

## Stack e Versoes

- GitHub Actions
- `actions/checkout@v4`
- `actions/setup-node@v4` (Node 20)
- `cypress-io/github-action@v6`
- `grafana/setup-k6-action@v1`
- `actions/upload-artifact@v4`

## Arquitetura da Pipeline

Gatilhos:

- `push` em `main`, `master`, `develop`, `feat/**`, `codex/**`
- `pull_request` para `main`, `master`, `develop`
- `workflow_dispatch` com inputs:
  - `run_performance` (utilizado)
  - `run_mobile` (declarado, sem job ativo nesta versao)

Jobs ativos:

- `api-tests`
  - instala dependencias em `automation/API`
  - executa `npm run test:api:all`
  - publica `api-evidence`

- `ui-tests`
  - executa Cypress em Chrome em `automation/UI`
  - publica `ui-evidence`

- `performance-tests`
  - instala k6
  - executa login + catalogo em `performance/k6`
  - publica `performance-evidence`

## Cenarios Cobertos

### API

- suite completa de API (`test:api:all`) com retry de 1 tentativa

### UI

- suite completa Cypress (`test:ui`)

### Performance

- `login-performance.js`
- `catalog-performance.js`

## Pre-requisitos

Para a pipeline executar com sucesso no GitHub, o repositorio precisa conter a estrutura esperada pelo workflow:

- `automation/API`
- `automation/UI`
- `performance/k6`
- `reports/ci`

Observacao importante:

- este repositorio atual contem apenas a definicao de CI/CD e nao inclui as camadas de teste acima.

## Instalacao

Nao ha instalacao obrigatoria para editar arquivos YAML.

Para simulacao local de workflow, ferramentas opcionais:

- `act` (nao instalado neste ambiente)
- parser YAML local (PowerShell `ConvertFrom-Yaml` ou Python `PyYAML`)

## Execucao da Pipeline

### Execucao no GitHub Actions

1. Abra a aba **Actions**.
2. Selecione **QA Complete Pipeline**.
3. Clique em **Run workflow**.
4. Defina `run_performance=true` quando quiser incluir k6.

### Execucao local

- a execucao real da pipeline local so e viavel quando a estrutura de pastas requerida existe no repositorio.
- neste repo isolado, os caminhos usados em `working-directory` nao existem.

## Estrutura do Projeto

```text
.
|-- .github/
|   `-- workflows/
|       `-- qa-pipeline.yml
|-- github-actions/
|   `-- qa-pipeline.yml
`-- README.md
```

## Explicacao do Teste neste Repo Isolado

Resultado do teste realizado neste repositorio:

- validacao estrutural: **OK**
  - arquivos de workflow e README presentes
- validacao de execucao da pipeline: **bloqueada por design**
  - paths exigidos no workflow nao existem aqui (`automation/*`, `performance/*`, `reports/ci`)
- validacao local de YAML: **nao concluida por falta de ferramenta no ambiente**
  - `ConvertFrom-Yaml` indisponivel
  - `PyYAML` nao instalado
- simulacao local de Actions: **nao disponivel**
  - `act` nao instalado

Conclusao:

- este repositorio funciona como repositorio de definicao/documentacao da pipeline.
- para execucao ponta a ponta, use o workflow no monorepo que contem as pastas de testes ou adapte o checkout para multiplos repositorios.

## Troubleshooting

### 1) Falha por path inexistente no workflow

Sintoma comum:

- erro de `working-directory` nao encontrado (`automation/API`, `automation/UI`, `performance/k6`)

Acoes recomendadas:

- executar em repositorio com a estrutura completa
- ou ajustar a pipeline para fazer checkout de repositorios separados

### 2) Nao valida sintaxe YAML localmente

Sintoma comum:

- ferramenta de parser nao encontrada

Acoes recomendadas:

- instalar PowerShell com `ConvertFrom-Yaml` disponivel
- ou instalar `PyYAML` para validacao via Python

### 3) Nao consegue simular GitHub Actions localmente

Sintoma comum:

- comando `act` indisponivel

Acoes recomendadas:

- instalar `act` e Docker
- usar validacao direta via GitHub Actions

## Boas Praticas para Evolucao

- Tratar `.github/workflows/qa-pipeline.yml` como fonte de verdade
- Manter o arquivo espelho sincronizado quando houver alteracoes
- Versionar toda alteracao de job com descricao clara no README
- Evitar referencias a paths inexistentes sem estrategia de checkout

## Autor

Lucas Viggiano

## Licenca

ISC
