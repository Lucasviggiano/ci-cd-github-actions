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

Este projeto centraliza pipelines de qualidade para execucao automatizada de testes por camada.

Workflows disponiveis neste repo:

- `.github/workflows/qa-pipeline.yml` (pipeline completa documental)
- `.github/workflows/tcc-api-smoke.yml` (fluxo executavel baseado no repo TCC)

Espelho documental:

- `github-actions/qa-pipeline.yml`
- `github-actions/tcc-api-smoke.yml`

## Stack e Versoes

- GitHub Actions
- `actions/checkout@v4`
- `actions/setup-node@v4` (Node 20)
- `cypress-io/github-action@v6`
- `grafana/setup-k6-action@v1`
- `actions/upload-artifact@v4`

## Arquitetura da Pipeline

### 1) `qa-pipeline.yml` (completa)

Gatilhos:

- `push` em `main`, `master`, `develop`, `feat/**`, `codex/**`
- `pull_request` para `main`, `master`, `develop`
- `workflow_dispatch` com inputs:
  - `run_performance` (utilizado)
  - `run_mobile` (declarado, sem job ativo nesta versao)

Jobs ativos:

- `api-tests`
- `ui-tests`
- `performance-tests`

### 2) `tcc-api-smoke.yml` (fluxo executavel neste repo)

Gatilho:

- `workflow_dispatch`

Job ativo:

- `api-smoke-from-tcc`
  - faz checkout deste repo
  - clona `https://github.com/Lucasviggiano/ebac-tcc-qe.git`
  - instala dependencias em `tcc-source/automation/API`
  - executa `npm run test:api:smoke`
  - publica artefato `api-smoke-evidence`

## Cenarios Cobertos

### `qa-pipeline.yml`

- API (suite completa)
- UI (suite Cypress)
- Performance (login + catalogo k6)

### `tcc-api-smoke.yml`

- API Smoke (GraphQL health) da base TCC

## Pre-requisitos

Para execucao com sucesso no GitHub:

- permissao de checkout e clone de repositorio publico
- disponibilidade do repositório base `Lucasviggiano/ebac-tcc-qe`
- (opcional) secrets/vars para sobrescrever credenciais e URLs

## Instalacao

Nao ha instalacao obrigatoria para editar os YAMLs.

## Execucao da Pipeline

### Fluxo recomendado neste repo (funcional)

1. Abra a aba **Actions**.
2. Selecione **TCC API Smoke Flow**.
3. Clique em **Run workflow**.
4. Execute e acompanhe o job `api-smoke-from-tcc`.

### Pipeline completa

- A `QA Complete Pipeline` permanece disponivel, mas depende da estrutura monorepo (`automation/*`, `performance/*`) no mesmo repositorio.

## Estrutura do Projeto

```text
.
|-- .github/
|   `-- workflows/
|       |-- qa-pipeline.yml
|       `-- tcc-api-smoke.yml
|-- github-actions/
|   |-- qa-pipeline.yml
|   `-- tcc-api-smoke.yml
`-- README.md
```

## Explicacao do Teste neste Repo Isolado

Status atual de testabilidade:

- `qa-pipeline.yml`: **nao executavel isoladamente**
  - referencia paths locais inexistentes neste repo (`automation/*`, `performance/*`, `reports/ci`)

- `tcc-api-smoke.yml`: **executavel**
  - resolve a limitacao ao clonar o repo base do TCC durante o job
  - executa ao menos um fluxo real de teste (`test:api:smoke`)

Conclusao:

- este repositorio agora possui pelo menos um fluxo de teste funcional baseado no TCC.

## Troubleshooting

### 1) Falha ao clonar repositório base

Sintoma comum:

- erro no passo `git clone` do job `api-smoke-from-tcc`

Acoes recomendadas:

- validar disponibilidade do GitHub no runner
- conferir se URL do repositorio base permanece publica

### 2) Falha na suite API smoke

Sintoma comum:

- job falha no passo `npm run test:api:smoke`

Acoes recomendadas:

- verificar indisponibilidade do ambiente `lojaebac.ebaconline.art.br`
- revisar logs em `api-smoke-evidence`

### 3) Pipeline completa falhando por path

Sintoma comum:

- erro de `working-directory` nao encontrado em `qa-pipeline.yml`

Acoes recomendadas:

- usar o fluxo `tcc-api-smoke.yml` neste repo isolado
- ou mover `qa-pipeline.yml` para monorepo com estrutura completa

## Boas Praticas para Evolucao

- Tratar `.github/workflows/qa-pipeline.yml` como baseline de pipeline completa
- Manter `tcc-api-smoke.yml` como fluxo minimo funcional para este repo isolado
- Sincronizar sempre os arquivos espelho em `github-actions/`
- Versionar alteracoes de pipeline com changelog resumido no README

## Autor

Lucas Viggiano

## Licenca

ISC
