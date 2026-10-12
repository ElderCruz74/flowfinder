# AI Handoff — FlowFinder

Documento curto para outra IA assumir o projeto sem contexto de chat.

## Objetivo

App corporativo de mapeamento de hábitos/rotina com escolha forçada X/Y, painel gestor e relatório preditivo (radar).

## O que já existe neste repositório

- Questionário oficial: `docs/habitus.json`
- Mapa de telas e DDL: `APP_MAP.md`
- Visão geral e roadmap: `README.md`
- Este handoff: `docs/AI_HANDOFF.md`

## O que existe no Grok Build (outro ambiente)

MVP funcional (TanStack Start):

- Colaborador: login, pendências, wizard 6 perguntas, envio
- Gestor: liberações, escolha de API, POST xAI → JSON
- Estado demo em `localStorage`

O **código-fonte TSX completo** pode ainda não estar neste repo. Se precisar do app rodando:

1. Reabra o projeto no Grok Build onde as etapas 1–7 foram executadas, **ou**
2. Reimplemente a partir de `APP_MAP.md` + `docs/habitus.json` + contratos do README.

## Próxima tarefa prioritária (etapa 8)

1. Consumir `report.dimensoes[]`
2. Renderizar **Roda de Fit Funcional** (radar)
3. Montar tela de relatório final no painel do gestor (resumo + scores + gráfico)

## Regras que não quebrar

- 6 perguntas, X e Y distintos, pesos 2 e 1
- Tags `internal_tag` só no backend/scoring
- Sem chaves de API no frontend
- Nomenclatura institucional FlowFinder / Mapeamento de Hábitos e Rotinas
- IA só sob clique do gestor

## Credenciais demo (preview)

- Colaborador: `colaborador` / `1234`
- Gestor: `gestor` / `1234`

## Repo

https://github.com/ElderCruz74/flowfinder
