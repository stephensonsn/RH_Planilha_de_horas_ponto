# Arquitetura Completa: Módulo de Timesheet e Aprovações PJ - Humanamente

**Projeto:** Humanamente - Substituição do Módulo de Horas e Ponto  
**Baseado em:** Análise Interativa da Demo Kimai  
**Perfil:** Colaboradores PJ (Pessoa Jurídica) e Gestores  
**Autor:** Manus AI  
**Data:** 01 de Fevereiro de 2026

---

## 🎯 SUMÁRIO EXECUTIVO

Este documento consolida **toda a arquitetura visual, funcional e técnica** para o novo módulo de apontamento e aprovação de horas do Humanamente, focado exclusivamente em **colaboradores PJ (Pessoa Jurídica)**. A solução é uma tradução fiel da experiência do **Kimai** para a stack do Humanamente (React, TypeScript, Supabase, Tailwind CSS).

### Visão Geral da Solução

A solução proposta abandona a integração direta com sistemas legados em favor de uma **tradução nativa** dos conceitos do Kimai para garantir performance, manutenibilidade e uma experiência de usuário coesa.

**Dois Perfis de Usuário:**
-   **Colaborador PJ:** Interface rica para apontamento de horas por projeto/atividade, com timer ativo, lançamento manual e cálculo automático de valores.
-   **Team Leader (Gestor):** Interface de gestão com visão da equipe, filtros avançados, fluxo de aprovação e ações em massa.

### Fluxo End-to-End

1.  **Apontamento:** Colaboradores PJ registram suas horas via timer ou lançamento manual.
2.  **Submissão:** Ao final do período, submetem os registros para aprovação (status muda de `draft` para `pending`).
3.  **Notificação:** Gestor é notificado por e-mail sobre as horas pendentes.
4.  **Aprovação:** Gestor acessa a tela de aprovação, analisa e aprova ou rejeita os registros.
5.  **Automação:**
    -   **Se Aprovado:** Um Pedido de Compra é criado automaticamente e o financeiro é notificado.
    -   **Se Rejeitado:** Colaborador é notificado com o motivo obrigatório.
    -   E-mails são disparados em cada etapa do processo.

---

## 1. ARQUITETURA VISUAL E WIREFRAMES

### 1.1 Estrutura Visual Global (Paleta de Cores Kimai)

-   **Fundo Principal:** `#2c3e50` (Azul-ardósia escuro)
-   **Sidebar/Header:** `#34495e` (Azul-ardósia mais claro)
-   **Ações Positivas (Criar, Aprovar):** `#27ae60` (Verde)
-   **Navegação/Links:** `#3498db` (Azul)
-   **Alertas/Timer Ativo:** `#e74c3c` (Vermelho)
-   **Filtros/Exportação:** `#f39c12` (Amarelo)
-   **Texto Principal:** `#ecf0f1` (Branco/Cinza claro)

### 1.2 Wireframe: Interface do Colaborador PJ

**Tela Principal: "Meus Horários"**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER: [▶️ Timer: 0:00] [🔄 Reiniciar] [👤 Avatar: João Silva (PJ)]         │
├─────────────────────────────────────────────────────────────────────────────┤
│ SIDEBAR │ Meus horários                                                     │
│         ├───────────────────────────────────────────────────────────────────┤
│ ☰ Menu  │ [🔲 Colunas] [🔽 Período] [🔍 Filtros] [Procurar...]    [+ Criar] [⬇ Exportar]│
│         ├───────────────────────────────────────────────────────────────────┤
│ • Dash  │ ┌─┬────┬───────┬─────────┬─────────┬─────────┬─────────┬─────────┐ │
│ • Tempo │ │✅│Data│Projeto│Atividade│Duração  │Valor    │Status   │Ações    │ │
│   - Meus│ ├─┼────┼───────┼─────────┼─────────┼─────────┼─────────┼─────────┤ │
│   - Cal │ │✅│01/02│Proj A │Design   │ 8:00    │ R$800   │ Rascunho│▶ ✏️ 🗑️ │ │
│         │ │✅│31/01│Proj B │Dev      │ 6:30    │ R$650   │ Rascunho│▶ ✏️ 🗑️ │ │
│         │ └─┴────┴───────┴─────────┴─────────┴─────────┴─────────┴─────────┘ │
│         │ ┌───────────────────────────────────────────────────────────────┐ │
│         │ │ 2 selecionados │ [Submeter para Aprovação] [Exportar] [Deletar]│ │
│         │ └───────────────────────────────────────────────────────────────┘ │
└─────────┴───────────────────────────────────────────────────────────────────┘
```

**Modal de Criação/Edição:**
```
┌───────────────────────────────────────────────────────────┐
│ Criar Registro de Tempo                            ? ×    │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  De *                Duração / Fim                        │
│  [📅 01/02/2026] [🕐 09:00]   [⏱ 8:00] [🕐 17:00]          │
│                                                           │
│  Projeto *                                                │
│  [🔽 Projeto A - Cliente XYZ                    ]          │
│                                                           │
│  Atividade *                                              │
│  [🔽 Desenvolvimento                            ]          │
│                                                           │
│  Descrição                                                │
│  [Implementação do módulo de autenticação...    ]         │
│  [                                              ]         │
│                                                           │
│  Valor/Hora: R$ 100,00    Faturável? [✅]                  │
│  Valor Total: R$ 800,00                                   │
│                                                           │
│                                    ┌────────┬──────────┐  │
│                                    │ Salvar │  Fechar  │  │
│                                    └────────┴──────────┘  │
└───────────────────────────────────────────────────────────┘
```

**Dashboard (Visão Geral):**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER: [▶️ Timer: 0:00] [🔄 Reiniciar] [👤 Avatar: João Silva (PJ)]         │
├─────────────────────────────────────────────────────────────────────────────┤
│ SIDEBAR │ Dashboard                                                         │
│         ├───────────────────────────────────────────────────────────────────┤
│         │ ┌───────────────────────────────────────────────────────────────┐ │
│         │ │ Seu horário de trabalho                     [<] [Semana 05] [>] │ │
│         │ │ ┌───────────────────────────────────────────────────────────┐ │ │
│         │ │ │ [Gráfico de Linha: Horas por Dia na Semana]               │ │ │
│         │ │ └───────────────────────────────────────────────────────────┘ │ │
│         │ └───────────────────────────────────────────────────────────────┘ │
│         │ ┌───────────┬───────────┬───────────┬───────────┐               │
│         │ │ 8:00      │ 40:00     │ 160:00    │ 1.920:00  │               │
│         │ │ Hoje      │ Semana 05 │ Fevereiro │ Ano 2026  │               │
│         │ └───────────┴───────────┴───────────┴───────────┘               │
│         │ ┌───────────┬───────────┬───────────┬───────────┐               │
│         │ │ R$ 800    │ R$ 4.000  │ R$ 16.000 │ R$ 192.000│               │
│         │ │ Hoje      │ Semana 05 │ Fevereiro │ Ano 2026  │               │
│         │ │ [👁️ Ocultar Valores]                          │               │
│         │ └───────────┴───────────┴───────────┴───────────┘               │
└─────────┴───────────────────────────────────────────────────────────────────┘
```

**Calendário (Visualização Mensal):**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Calendário                                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│  fevereiro de 2026    [Mês] [Semana] [Dia]    [Hoje]    [<] [>]           │
├────┬────────┬────────┬────────┬────────┬────────┬────────┬────────┤       │
│    │ seg.   │ ter.   │ qua.   │ qui.   │ sex.   │ sáb.   │ dom.   │       │
├────┼────────┼────────┼────────┼────────┼────────┼────────┼────────┤       │
│ Sm5│   26   │   27   │   28   │   29   │   30   │   31   │    1   │       │
│    │        │        │        │        │        │[🟦 6:30]│        │       │
│    │        │        │        │        │        │ Proj B │        │       │
├────┼────────┼────────┼────────┼────────┼────────┼────────┼────────┤       │
│ Sm6│    2   │    3   │    4   │    5   │    6   │    7   │    8   │       │
│    │[🟩 8:00]│        │        │        │        │        │        │       │
│    │ Proj A │        │        │        │        │        │        │       │
└────┴────────┴────────┴────────┴────────┴────────┴────────┴────────┘       │
```

### 1.3 Wireframe: Interface do Team Leader (Aprovação)

**Tela Principal: "Aprovação de Horas" (All times)**
```
┌───────────────────────────────────────────────────────────────────────────────────────────┐
│ HEADER: [👤 Avatar: Tony Maier (Gestor)]                                                  │
├───────────────────────────────────────────────────────────────────────────────────────────┤
│ SIDEBAR │ Aprovação de Horas                                                              │
│         ├─────────────────────────────────────────────────────────────────────────────────┤
│ ☰ Menu  │ [Período: Últimos 7 dias▼] [Status: Pendente▼] [Usuário: Todos▼]  🔍           │
│         │                                                   [+ Criar] [Exportar▼]         │
│ • Dash  ├─────────────────────────────────────────────────────────────────────────────────┤
│ • All   │ ☑️ │ Usuário      │ Data     │ Projeto │ Atividade│ Horas │ Valor   │ Status │ Ações │
│   times │───┼──────────────┼──────────┼─────────┼──────────┼───────┼─────────┼────────┼───────│
│ • Export│ ☑ │ João Silva   │ 01/02/26 │ Proj A  │ Design   │ 8:00  │ R$800   │Pendente│ ✓ ✗ ✎ │
│ • Report│ ☑ │ João Silva   │ 31/01/26 │ Proj B  │ Dev      │ 6:30  │ R$650   │Pendente│ ✓ ✗ ✎ │
│         │ ☑ │ Pedro Costa  │ 31/01/26 │ Proj A  │ Design   │ 10:00 │ R$1.200 │Pendente│ ✓ ✗ ✎ │
│         ├─────────────────────────────────────────────────────────────────────────────────┤
│         │ 3 selecionados │ [Aprovar Selecionados] [Rejeitar Selecionados] [Exportar]      │
└─────────┴─────────────────────────────────────────────────────────────────────────────────┘
```

**Modal de Rejeição:**
```
┌──────────────────────────────────┐
│ Rejeitar Registro de Hora    ×   │
├──────────────────────────────────┤
│ Usuário: João Silva              │
│ Data: 01/02/26 - 8:00h           │
│ Projeto: Projeto A               │
│                                  │
│ Motivo da Rejeição (obrigatório) │
│ [______________________________] │
│ [                              ] │
│ [                              ] │
│                                  │
│      [Cancelar] [Confirmar]      │
└──────────────────────────────────┘
```

---

## 2. ARQUITETURA TÉCNICA

### 2.1 Estrutura do Banco de Dados (Supabase - PostgreSQL)

**Tabela `profiles` (atualizar existente):**
```sql
ALTER TABLE profiles 
ADD COLUMN hourly_rate NUMERIC(10, 2),
ADD COLUMN team_id UUID REFERENCES teams(id),
ADD COLUMN show_financial_data BOOLEAN DEFAULT false;
```

**Tabela `projects`:**
```sql
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  client_name TEXT,
  color TEXT DEFAULT '#3498db',
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_projects_active ON projects(is_active);
```

**Tabela `activities`:**
```sql
CREATE TABLE activities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_activities_project ON activities(project_id);
```

**Tabela `timesheet_entries` (principal):**
```sql
CREATE TABLE timesheet_entries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) NOT NULL,
  project_id UUID REFERENCES projects(id) NOT NULL,
  activity_id UUID REFERENCES activities(id) NOT NULL,
  start_time TIMESTAMPTZ NOT NULL,
  end_time TIMESTAMPTZ,
  duration INT, -- Em segundos
  description TEXT,
  is_billable BOOLEAN DEFAULT true,
  hourly_rate NUMERIC(10, 2), -- Valor/hora no momento do registro
  total_value NUMERIC(10, 2), -- Calculado: (duration/3600) * hourly_rate
  status TEXT NOT NULL DEFAULT 'draft', -- draft, pending, approved, rejected
  rejection_reason TEXT,
  approved_by UUID REFERENCES profiles(id),
  approved_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_timesheet_user ON timesheet_entries(user_id);
CREATE INDEX idx_timesheet_status ON timesheet_entries(status);
CREATE INDEX idx_timesheet_dates ON timesheet_entries(start_time, end_time);
```

**Tabela `purchase_orders`:**
```sql
CREATE TABLE purchase_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) NOT NULL,
  total_amount NUMERIC(10, 2) NOT NULL,
  total_hours NUMERIC(10, 2) NOT NULL,
  period_start DATE NOT NULL,
  period_end DATE NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending_payment', -- pending_payment, paid
  invoice_requested_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_purchase_orders_user ON purchase_orders(user_id);
CREATE INDEX idx_purchase_orders_status ON purchase_orders(status);
```

**Tabela `timesheet_edits` (Auditoria):**
```sql
CREATE TABLE timesheet_edits (
  id BIGSERIAL PRIMARY KEY,
  entry_id UUID REFERENCES timesheet_entries(id) NOT NULL,
  edited_by UUID REFERENCES profiles(id) NOT NULL,
  edited_at TIMESTAMPTZ DEFAULT now(),
  previous_data JSONB NOT NULL,
  new_data JSONB NOT NULL
);

CREATE INDEX idx_timesheet_edits_entry ON timesheet_edits(entry_id);
```

**Trigger de Auditoria:**
```sql
CREATE OR REPLACE FUNCTION log_timesheet_edit()
RETURNS TRIGGER AS $$
BEGIN
  IF (OLD.* IS DISTINCT FROM NEW.*) THEN
    INSERT INTO timesheet_edits (entry_id, edited_by, previous_data, new_data)
    VALUES (
      NEW.id,
      auth.uid(),
      to_jsonb(OLD),
      to_jsonb(NEW)
    );
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER timesheet_edit_trigger
AFTER UPDATE ON timesheet_entries
FOR EACH ROW
EXECUTE FUNCTION log_timesheet_edit();
```

### 2.2 Componentes React Principais

**1. `TimesheetDashboard.tsx`** (Colaborador PJ)
- Dashboard com gráficos de horas e valores
- Cards de resumo (hoje, semana, mês, ano)
- Toggle para mostrar/ocultar valores financeiros

**2. `TimesheetTable.tsx`** (Colaborador PJ)
- Tabela principal de registros
- Filtros por período, projeto, atividade, status
- Ações em massa (submeter, exportar, deletar)
- Integração com modal de criação/edição

**3. `TimesheetModal.tsx`** (Colaborador PJ)
- Modal para criar/editar registros
- Validações de campos obrigatórios
- Cálculo automático de valor total
- Suporte a timer ativo (sem end_time)

**4. `TimesheetCalendar.tsx`** (Colaborador PJ)
- Visualização em calendário (mês/semana/dia)
- Eventos coloridos por projeto
- Drag-and-drop para mover registros
- Click para criar/editar

**5. `ApprovalDashboard.tsx`** (Team Leader)
- Tabela de aprovação com visão da equipe
- Filtros avançados (usuário, status, período, projeto)
- Ações em massa (aprovar, rejeitar, exportar)
- Modal de rejeição com motivo obrigatório

**6. `TimerWidget.tsx`** (Header Global)
- Timer ativo no header
- Botão play/stop
- Integração com modal de criação

### 2.3 Automações (Supabase Edge Functions)

**1. `on-timesheet-submitted`**
```typescript
// Trigger: UPDATE na timesheet_entries com status mudando para 'pending'
// Ação: Envia e-mail para o gestor da equipe

import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  const { record } = await req.json()
  
  // Buscar gestor da equipe do usuário
  const supabase = createClient(...)
  const { data: user } = await supabase
    .from('profiles')
    .select('team_id, teams(manager_id)')
    .eq('id', record.user_id)
    .single()
  
  // Enviar e-mail para o gestor
  await sendEmail({
    to: user.teams.manager_email,
    subject: 'Novas horas pendentes de aprovação',
    body: `O colaborador ${user.name} submeteu ${record.duration/3600}h para aprovação.`
  })
  
  return new Response('OK', { status: 200 })
})
```

**2. `on-timesheet-approved`**
```typescript
// Trigger: UPDATE na timesheet_entries com status mudando para 'approved'
// Ação: Cria Pedido de Compra e envia e-mails

serve(async (req) => {
  const { record } = await req.json()
  
  const supabase = createClient(...)
  
  // Buscar todos os registros aprovados do usuário no período
  const { data: entries } = await supabase
    .from('timesheet_entries')
    .select('*')
    .eq('user_id', record.user_id)
    .eq('status', 'approved')
    .gte('start_time', startOfMonth)
    .lte('start_time', endOfMonth)
  
  // Calcular totais
  const totalHours = entries.reduce((sum, e) => sum + e.duration/3600, 0)
  const totalAmount = entries.reduce((sum, e) => sum + e.total_value, 0)
  
  // Criar Pedido de Compra
  await supabase.from('purchase_orders').insert({
    user_id: record.user_id,
    total_hours: totalHours,
    total_amount: totalAmount,
    period_start: startOfMonth,
    period_end: endOfMonth
  })
  
  // Enviar e-mails
  await sendEmail({ to: user.email, subject: 'Horas aprovadas', ... })
  await sendEmail({ to: finance.email, subject: 'Novo pedido de compra', ... })
  
  return new Response('OK', { status: 200 })
})
```

**3. `on-timesheet-rejected`**
```typescript
// Trigger: UPDATE na timesheet_entries com status mudando para 'rejected'
// Ação: Envia e-mail para o colaborador com o motivo

serve(async (req) => {
  const { record } = await req.json()
  
  await sendEmail({
    to: user.email,
    subject: 'Registro de hora rejeitado',
    body: `Seu registro foi rejeitado. Motivo: ${record.rejection_reason}`
  })
  
  return new Response('OK', { status: 200 })
})
```

---

## 3. FLUXO DE DADOS E REGRAS DE NEGÓCIO

### 3.1 Apontamento de Horas (Colaborador PJ)

**Método 1: Timer Ativo**
1. Colaborador clica no botão Play (▶️) no header
2. Modal abre para selecionar Projeto e Atividade
3. Colaborador clica em "Salvar" sem preencher Duração/Fim
4. Timer começa a contar no header
5. Ao clicar em Stop (⏹️), o modal reabre com a duração calculada
6. Colaborador confirma e o registro é salvo com status `draft`

**Método 2: Lançamento Manual**
1. Colaborador acessa "Meus horários"
2. Clica em "+ Criar"
3. Preenche todos os campos (Data, Hora Início, Duração/Fim, Projeto, Atividade)
4. Sistema calcula automaticamente o valor total (duração × valor/hora)
5. Colaborador clica em "Salvar"
6. Registro é criado com status `draft`

### 3.2 Submissão para Aprovação

1. Colaborador seleciona múltiplos registros com status `draft`
2. Clica em "Submeter para Aprovação"
3. Sistema valida:
   - Todos os campos obrigatórios preenchidos
   - Sem sobreposição de horários
   - Projeto e atividade ativos
4. Status muda para `pending`
5. Registros se tornam não editáveis para o colaborador
6. Edge Function dispara e-mail para o gestor

### 3.3 Aprovação (Team Leader)

**Aprovação Individual:**
1. Gestor acessa "Aprovação de Horas"
2. Filtra por status `pending`
3. Analisa o registro (usuário, projeto, duração, valor)
4. Clica no ícone ✓ (Aprovar)
5. Sistema valida (sem sobreposição, projeto ativo)
6. Status muda para `approved`
7. Edge Function cria Pedido de Compra e envia e-mails

**Aprovação em Massa:**
1. Gestor seleciona múltiplos registros via checkbox
2. Clica em "Aprovar Selecionados"
3. Sistema processa todos os registros
4. Edge Functions disparam para cada aprovação

**Rejeição:**
1. Gestor clica no ícone ✗ (Rejeitar)
2. Modal abre pedindo motivo (obrigatório)
3. Gestor digita o motivo e confirma
4. Status muda para `rejected`
5. Edge Function envia e-mail ao colaborador com o motivo

**Edição pelo Gestor:**
1. Gestor clica no ícone ✎ (Editar)
2. Modal abre pré-preenchido
3. Gestor corrige informações (duração, descrição, etc.)
4. Salva alterações
5. Trigger de auditoria registra a edição na tabela `timesheet_edits`

### 3.4 Geração de Pedido de Compra

- Ocorre automaticamente após aprovação das horas
- Consolida todas as horas aprovadas do usuário no período (mês)
- Cria um único pedido com:
  - Total de horas
  - Valor total
  - Período (início e fim do mês)
- Envia e-mail para o financeiro com os detalhes

### 3.5 Solicitação de Nota Fiscal

1. Gestor ou financeiro acessa a tela de Pedidos de Compra
2. Clica em "Solicitar NF" para um pedido específico
3. Sistema registra a data da solicitação (`invoice_requested_at`)
4. Edge Function envia e-mail ao colaborador PJ solicitando a emissão da NF

---

## 4. VALIDAÇÕES E REGRAS

### 4.1 Validações de Registro

- **Campos Obrigatórios:** Data, Hora Início, Projeto, Atividade
- **Duração ou Fim:** Pelo menos um deve ser preenchido (exceto timer ativo)
- **Sobreposição:** Não é permitido criar registros com horários sobrepostos para o mesmo usuário
- **Projeto/Atividade Ativos:** Apenas projetos e atividades ativas podem ser selecionados
- **Data Futura:** Não é permitido criar registros com data futura

### 4.2 Permissões (RLS - Row-Level Security)

**Colaborador PJ:**
- Pode ver, criar, editar e deletar apenas seus próprios registros com status `draft`
- Pode ver seus próprios registros com status `pending`, `approved` ou `rejected` (somente leitura)

**Team Leader:**
- Pode ver todos os registros dos membros da sua equipe
- Pode aprovar, rejeitar e editar registros com status `pending`
- Não pode ver registros de outras equipes

**Admin:**
- Pode ver e gerenciar todos os registros do sistema

### 4.3 Cálculos Automáticos

- **Duração:** Se apenas "Fim" for preenchido, a duração é calculada automaticamente (Fim - Início)
- **Fim:** Se apenas "Duração" for preenchida, o fim é calculado automaticamente (Início + Duração)
- **Valor Total:** Sempre calculado automaticamente: (duração em horas) × (valor/hora)

---

## 5. CHECKLIST DE IMPLEMENTAÇÃO

### Fase 1: Backend (Supabase)

- [ ] Criar migrations para todas as tabelas (projects, activities, timesheet_entries, purchase_orders, timesheet_edits)
- [ ] Atualizar tabela `profiles` com novos campos (hourly_rate, team_id, show_financial_data)
- [ ] Implementar RLS (Row-Level Security) para hierarquia de equipes
- [ ] Criar trigger de auditoria para `timesheet_edits`
- [ ] Desenvolver Edge Function `on-timesheet-submitted`
- [ ] Desenvolver Edge Function `on-timesheet-approved`
- [ ] Desenvolver Edge Function `on-timesheet-rejected`
- [ ] Configurar serviço de e-mail (Resend ou SendGrid)

### Fase 2: Frontend (React)

- [ ] Desenvolver `TimesheetDashboard.tsx` (colaborador)
- [ ] Desenvolver `TimesheetTable.tsx` (colaborador)
- [ ] Desenvolver `TimesheetModal.tsx` (criar/editar)
- [ ] Desenvolver `TimesheetCalendar.tsx` (visualização)
- [ ] Desenvolver `TimerWidget.tsx` (header)
- [ ] Desenvolver `ApprovalDashboard.tsx` (gestor)
- [ ] Desenvolver modal de rejeição com campo de motivo obrigatório
- [ ] Implementar filtros avançados (período, projeto, atividade, status)
- [ ] Implementar ações em massa (submeter, aprovar, rejeitar, exportar)
- [ ] Integrar todas as chamadas de API do Supabase

### Fase 3: Integrações e Testes

- [ ] Testar fluxo end-to-end completo (apontamento → submissão → aprovação → pedido de compra)
- [ ] Validar todas as regras de negócio (sobreposição, campos obrigatórios, etc.)
- [ ] Testar envio de e-mails em todas as etapas
- [ ] Validar cálculos automáticos (duração, valor total)
- [ ] Testar RLS (colaborador não vê dados de outros, gestor só vê sua equipe)
- [ ] Testar auditoria de edições
- [ ] Realizar testes de performance com grande volume de dados
- [ ] Testar exportação de dados (PDF, Excel, CSV)

---

## 6. CONCLUSÃO

Esta arquitetura fornece um plano de ação completo e detalhado para a implementação do módulo de timesheet focado em colaboradores PJ no Humanamente. A solução é uma tradução fiel da experiência do Kimai, adaptada à stack tecnológica do projeto, garantindo performance, escalabilidade e uma experiência de usuário de alta qualidade.

**Diferenciais da Solução:**
- ✅ Fidelidade visual total ao Kimai
- ✅ Banco de dados robusto com índices otimizados e auditoria
- ✅ Componentes React modulares e reutilizáveis
- ✅ Automações completas via Edge Functions
- ✅ Fluxo de aprovação com validações e notificações
- ✅ Geração automática de Pedidos de Compra
- ✅ Segurança via RLS (Row-Level Security)
- ✅ Escalável e pronto para crescimento
