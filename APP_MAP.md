# APP_MAP — FlowFinder

Mapa técnico para implementação e para outras IAs.

## Telas

### Colaborador

| ID | Nome | Função |
|----|------|--------|
| C1 | Login | Identificação (demo: login/senha) |
| C2 | Dashboard de pendências | Card do mapeamento se `status=pending` |
| C3 | Wizard | 6 perguntas, X/Y, barra de progresso |
| C4 | Agradecimento | Confirma envio; opcionalmente mostra payload |

### Gestor

| ID | Nome | Função |
|----|------|--------|
| G1 | Login gestor | Acesso administrativo |
| G2 | Liberações | Lista equipe; liberar / revogar; ver concluídos |
| G3 | API de IA | Seleção do provedor (xAI no MVP) |
| G4 | Relatório | Dispara POST, recebe JSON; (futuro) radar |

## Fluxo de dados

```
Gestor libera assigned_assessments (pending)
    → Colaborador vê C2 e responde C3
    → responses.answers_json + status completed
    → Gestor dispara requestHabitusReport(answers)
    → reports.json (dimensoes + resumo)
    → UI radar (etapa 8)
```

## DDL sugerida (PostgreSQL / Supabase)

```sql
-- users
create table users (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  email text unique,
  cpf text unique,
  role text not null check (role in ('employee', 'admin')),
  created_at timestamptz default now()
);

-- assigned_assessments
create table assigned_assessments (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references users(id),
  assessment_id text not null default 'habitus_v1',
  status text not null check (status in ('locked', 'pending', 'completed')),
  released_at timestamptz,
  completed_at timestamptz,
  created_at timestamptz default now()
);

-- responses
create table responses (
  id uuid primary key default gen_random_uuid(),
  assigned_assessment_id uuid not null references assigned_assessments(id),
  user_id uuid not null references users(id),
  answers_json jsonb not null,
  created_at timestamptz default now()
);

-- reports
create table reports (
  id uuid primary key default gen_random_uuid(),
  response_id uuid not null references responses(id),
  provider text not null, -- xai | openai | anthropic | gemini
  report_json jsonb not null,
  created_at timestamptz default now()
);
```

### RLS (orientação)

- `employee`: `select/insert` apenas onde `user_id = auth.uid()`
- `admin`: CRUD em liberações e leitura de responses/reports

## Regras de negócio

1. Só avança no wizard com X e Y preenchidos e distintos.
2. Assessment `completed` não pode ser refeito pelo colaborador.
3. Gestor só gera relatório se existirem 6 respostas salvas.
4. Chamada de IA é sob demanda (botão), não automática.
5. Tags internas (`internal_tag`) nunca na UI do colaborador.

## Dimensões do radar (contrato estável)

| id | label |
|----|--------|
| adaptacao | Adaptação |
| ritmo | Ritmo e meta |
| persistencia | Persistência |
| etica | Ética na negociação |
| fluencia_social | Fluência social |
| gestao_de_conflito | Gestão de conflito |

## Storage keys do MVP (browser)

- `flowfinder.habitus.v1` — sessão colaborador (nome, answers, completedAt)
- `flowfinder.gestor.v1` — board: people, aiProvider, payloads

## Endpoint IA (MVP)

```
POST https://api.x.ai/v1/chat/completions
Authorization: Bearer $XAI_API_KEY
Body: { model: "grok-4.5", temperature: 0.2, max_tokens: 500, messages: [...] }
```

Handler: `requestHabitusReport` em `src/lib/habitus-report.ts`.
