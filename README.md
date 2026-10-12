# FlowFinder — Mapeamento de Hábitos e Rotinas

Solução corporativa de mapeamento comportamental de colaboradores com metodologia de **Escolha Forçada [X] / [Y]**, painel do gestor e geração de relatório via IA (Roda de Fit Funcional).

> Documentação orientada a **humanos e a outras IAs** que forem continuar o produto.

Repo: https://github.com/ElderCruz74/flowfinder

---

## Visão do produto

| Papel | O que faz |
|-------|-----------|
| **Colaborador** | Login → vê pendências → responde 6 situações (X principal + Y secundária) → envia |
| **Gestor** | Libera mapeamentos → escolhe API de IA → gera JSON do relatório |

Nome institucional: **Mapeamento de Hábitos e Rotinas** (não "teste psicológico").

---

## Estado atual (MVP no Grok Build)

Implementado em etapas:

1. Metodologia X/Y + questionário (6 situações)
2. Nomenclatura FlowFinder
3. Validação X ≠ Y
4. Modelagem conceitual de dados
5. Telas do colaborador (C1 Login, C2 Dashboard, C3 Wizard)
6. Painel do gestor (liberações + seleção de API)
7. POST para IA (xAI Grok) devolvendo JSON
8. *(próximo)* Gráfico de radar + relatório final visual

### Credenciais de demonstração (só no preview)

| Papel | Login | Senha |
|-------|-------|-------|
| Colaborador | `colaborador` | `1234` |
| Gestor | `gestor` | `1234` |

---

## Arquitetura real do MVP (Grok Build)

O app construído no Grok Build **não** é low-code clássico nem Supabase obrigatório. Stack do sandbox:

| Camada | Tecnologia |
|--------|------------|
| Frontend | TanStack Start + React + Tailwind |
| Estado demo | `localStorage` (sem auth real de produção) |
| IA | `createServerFn` → `POST https://api.x.ai/v1/chat/completions` (modelo `grok-4.5`) |
| Chave | `XAI_API_KEY` (servidor, injetada pelo ambiente) |

Rotas principais:

- `/` — fluxo do colaborador
- `/gestor` — painel do avaliador

Arquivos-chave (quando o código-fonte for versionado a partir do Grok Build):

```
src/data/habitus.json          # 6 perguntas oficiais
src/lib/releases.ts            # liberações + payloads + provedor de IA
src/lib/habitus-report.ts      # server fn POST → JSON da IA
src/routes/index.tsx           # login + home + wizard + thanks
src/routes/gestor.tsx          # painel gestor
```

---

## Questionário (contrato)

- **6 perguntas** (Parte 1: q1–q3; Parte 2: q4–q6)
- Cada uma: **4 alternativas** (códigos A–D / E–H / I–L)
- Usuário marca **X** (peso 2) e **Y** (peso 1)
- **X ≠ Y** obrigatório
- Tags internas da Parte 2 **não** são mostradas ao colaborador; entram no payload para scoring

Fonte canônica: [`docs/habitus.json`](./docs/habitus.json).

---

## Payload de respostas (exemplo)

```json
{
  "assessment_id": "habitus_v1",
  "respondent": "Ana Ribeiro",
  "completed_at": "2026-10-12T00:00:00.000Z",
  "answers": [
    {
      "question_id": "q1",
      "choice_x": "B",
      "choice_y": "A",
      "weight_x": 2,
      "weight_y": 1,
      "tag_x": null,
      "tag_y": null
    }
  ]
}
```

---

## JSON esperado da IA (relatório)

```json
{
  "resumo": "texto curto",
  "dimensoes": [
    { "id": "adaptacao", "label": "Adaptação", "score": 0 },
    { "id": "ritmo", "label": "Ritmo e meta", "score": 0 },
    { "id": "persistencia", "label": "Persistência", "score": 0 },
    { "id": "etica", "label": "Ética na negociação", "score": 0 },
    { "id": "fluencia_social", "label": "Fluência social", "score": 0 },
    { "id": "gestao_de_conflito", "label": "Gestão de conflito", "score": 0 }
  ]
}
```

Scores: inteiros **0–100**. IDs fixos como acima. A **Roda de Fit Funcional** (radar) usa esse array.

---

## Modelo de dados alvo (produção)

Ver [`APP_MAP.md`](./APP_MAP.md). Resumo de tabelas:

- `users` — colaboradores e gestores
- `assigned_assessments` — liberações (`locked` / `pending` / `completed`)
- `responses` — `answers_json` + metadados
- `reports` — JSON da IA + status de renderização

RLS sugerida: role `employee` só vê o próprio usuário/respostas; `admin` gerencia liberações e relatórios.

---

## Variáveis de ambiente (produção)

```env
# Banco (quando sair do localStorage)
DATABASE_URL=
SUPABASE_URL=
SUPABASE_ANON_KEY=

# IA — no MVP Grok Build usa só xAI
XAI_API_KEY=

# Opcionais (roadmap multi-modelo)
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
GEMINI_API_KEY=
```

**Nunca** commitar chaves reais. Nunca expor `XAI_API_KEY` no cliente.

---

## Instruções para outra IA continuar o trabalho

1. Ler este `README.md` + `APP_MAP.md` + `docs/habitus.json` + `docs/AI_HANDOFF.md`.
2. Não reescrever o questionário sem alinhamento: o texto em `habitus.json` é a versão oficial.
3. Manter validação **X ≠ Y** e pesos 2 / 1.
4. Etapa 8 pendente: renderizar radar a partir de `dimensoes[]` (Chart.js, Recharts ou SVG).
5. Se migrar para Supabase: implementar DDL de `APP_MAP.md` e trocar `localStorage` por queries autenticadas.
6. Chamadas de IA só no **servidor**, sob ação explícita do gestor (não no page load).
7. Nomenclatura: sempre **FlowFinder** / **Mapeamento de Hábitos e Rotinas**.

---

## Roadmap

| # | Item | Status |
|---|------|--------|
| 1–5 | Metodologia, telas colaborador | Feito (MVP) |
| 6 | Painel gestor + liberações + escolha de API | Feito (MVP) |
| 7 | POST IA → JSON | Feito (xAI) |
| 8 | Radar + relatório visual final | Pendente |
| 9 | Auth real + Postgres/Supabase | Pendente |
| 10 | Multi-modelo (OpenAI / Claude / Gemini) além de xAI | Pendente |

---

## Licença / uso

Uso interno corporativo. Ajuste a licença conforme a política da organização.
