# Análise Completa: Fluxo de Aprovação (Team Leader) - Kimai para Humanamente

**Usuário:** Tony Maier (tony_teamlead) - Head of Sales  
**Baseado em:** Demo Interativa Kimai (https://demo-empty.kimai.org)  
**Data da Análise:** 01 de Fevereiro de 2026

---

## 🎯 Sumário Executivo

Esta documentação detalha a **interface e o fluxo de aprovação de horas** do perfil **Team Leader** (Gestor) no Kimai. O objetivo é traduzir fielmente essa experiência para o Humanamente, completando o ciclo que começa com o apontamento do colaborador (PJ) e termina com a aprovação do gestor.

### Fluxo End-to-End (Visão Geral)

1.  **Colaborador (PJ):** Acessa a interface inspirada no Kimai, aponta horas (via timer ou manual) e submete para aprovação. Os registros mudam de `draft` para `pending`.
2.  **Notificação:** Uma Edge Function do Supabase dispara um e-mail para o Team Leader, informando que há horas pendentes.
3.  **Team Leader:** Acessa a tela **"Todas as Horas"** (All times), filtra por status `pending` e visualiza os registros da sua equipe.
4.  **Aprovação/Rejeição:** O gestor pode aprovar ou rejeitar os registros individualmente ou em massa.
5.  **Resultado:**
    *   **Se Aprovado:** O status muda para `approved`. Outra Edge Function dispara, criando um **Pedido de Compra** para o PJ e notificando o financeiro.
    *   **Se Rejeitado:** O status muda para `rejected`. O colaborador é notificado por e-mail com o motivo da rejeição (obrigatório).

---

## 1. Wireframe e Estrutura Visual: Tela de Aprovação ("All times")

Esta é a central de comando do gestor. A análise completa desta tela está no documento `analise_teamlead_all_times.md`, mas o wireframe adaptado para o Humanamente é o seguinte:

### Wireframe Adaptado (com Dados e Filtros)

```
┌───────────────────────────────────────────────────────────────────────────────────────────┐
│ Tela: Aprovação de Horas                                                                  │
├───────────────────────────────────────────────────────────────────────────────────────────┤
│  Filtros: [Período: Últimos 7 dias▼] [Status: Pendente▼] [Usuário: Todos▼] [Tipo: PJ▼]  🔍 │
│                                                         [+ Criar] [Exportar▼]             │
├───────────────────────────────────────────────────────────────────────────────────────────┤
│ ☑️ │ Usuário      │ Data     │ Projeto │ Horas │ Tipo │ Valor   │ Status   │ Ações         │
│────┼──────────────┼──────────┼─────────┼───────┼──────┼─────────┼──────────┼───────────────│
│ ☑  │ João Silva   │ 01/02/26 │ Proj A  │ 8:00  │ PJ   │ R$800   │ Pendente │ [✓] [✗] [✎] │
│ ☑  │ Maria Santos │ 01/02/26 │ Proj B  │ 6:30  │ CLT  │ —       │ Pendente │ [✓] [✗] [✎] │
│ ☑  │ Pedro Costa  │ 31/01/26 │ Proj A  │ 10:00 │ PJ   │ R$1.200 │ Pendente │ [✓] [✗] [✎] │
└───────────────────────────────────────────────────────────────────────────────────────────┘
│ Ações em Massa: [Aprovar Selecionados] [Rejeitar Selecionados] [Exportar Selecionados]     │
└───────────────────────────────────────────────────────────────────────────────────────────┘
```

### Legenda de Ações:
- **[✓] Aprovar:** Muda o status para `approved`.
- **[✗] Rejeitar:** Abre modal para inserir motivo obrigatório e muda o status para `rejected`.
- **[✎] Editar:** Abre modal para corrigir o registro antes de aprovar.

---

## 2. Detalhamento do Fluxo de Aprovação

### Passo 1: Acesso e Filtragem
- O Team Leader acessa a tela de aprovações.
- O filtro padrão deve ser **Status: Pendente** para focar na tarefa principal.
- Ele pode filtrar por usuário, projeto, período, etc.

### Passo 2: Análise dos Registros
- O gestor analisa a tabela, verificando a consistência dos dados: usuário, projeto, duração e valor (para PJ).
- A coluna "Valor" é crucial para o controle de custos.

### Passo 3: Decisão (Aprovar, Rejeitar, Editar)

**Aprovação:**
- Clica no ícone de Aprovar (✓) ou seleciona múltiplos e clica em "Aprovar Selecionados".
- **Regra de Negócio:** O sistema deve validar se não há sobreposições de horário para aquele usuário antes de confirmar a aprovação.
- **Automação (Supabase Edge Function):**
  1.  Muda o status do `timesheet_entry` para `approved`.
  2.  **Se `contract_type` for 'PJ':**
      -   Insere um novo registro na tabela `purchase_orders` com os dados consolidados (usuário, valor total, período).
      -   Envia um e-mail para o financeiro com os detalhes do pedido de compra.
  3.  Envia um e-mail de notificação para o colaborador.

**Rejeição:**
- Clica no ícone de Rejeitar (✗).
- **Modal de Motivo (Obrigatório):**
  ```
  ┌──────────────────────────────────┐
  │ Rejeitar Registro de Hora        │
  ├──────────────────────────────────┤
  │ Usuário: João Silva              │
  │ Data: 01/02/26 - 8:00h           │
  │                                  │
  │ Motivo da Rejeição (obrigatório) │
  │ [______________________________] │
  │                                  │
  │      [Cancelar] [Confirmar]      │
  └──────────────────────────────────┘
  ```
- **Automação (Supabase Edge Function):**
  1.  Muda o status do `timesheet_entry` para `rejected`.
  2.  Salva o motivo da rejeição no campo `rejection_reason` da tabela.
  3.  Envia um e-mail para o colaborador com o motivo claro.

**Edição:**
- Clica no ícone de Editar (✎).
- O mesmo modal de criação de registro abre, pré-preenchido.
- O gestor pode corrigir a descrição, o projeto, a duração, etc.
- **Regra de Negócio:** Deve haver um log de auditoria (`timesheet_edits`) registrando quem alterou, o quê e quando.

---

## 3. Integração com a Documentação Existente

Esta análise do fluxo do Team Leader complementa a documentação anterior. As principais atualizações necessárias são:

### 3.1 Atualização do Banco de Dados (`arquitetura_banco_dados.md`)

**Tabela `timesheet_entries`:**
- Adicionar campo `status` (enum: `draft`, `pending`, `approved`, `rejected`) com valor padrão `draft`.
- Adicionar campo `rejection_reason` (text, nullable).
- Adicionar campo `approved_by` (uuid, foreign key para `profiles`), para registrar quem aprovou.
- Adicionar campo `approved_at` (timestamp).

**Nova Tabela `purchase_orders`:**
```sql
CREATE TABLE purchase_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) NOT NULL,
  total_amount NUMERIC(10, 2) NOT NULL,
  period_start DATE NOT NULL,
  period_end DATE NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending_payment',
  created_at TIMESTAMPTZ DEFAULT now(),
  invoice_requested_at TIMESTAMPTZ
);
-- RLS para que gestores vejam os pedidos da sua equipe
```

**Nova Tabela `timesheet_edits` (Auditoria):**
```sql
CREATE TABLE timesheet_edits (
  id BIGSERIAL PRIMARY KEY,
  entry_id UUID REFERENCES timesheet_entries(id) NOT NULL,
  edited_by UUID REFERENCES profiles(id) NOT NULL,
  edited_at TIMESTAMPTZ DEFAULT now(),
  previous_data JSONB NOT NULL,
  new_data JSONB NOT NULL
);
-- Trigger para popular esta tabela em cada UPDATE da timesheet_entries
```

### 3.2 Atualização dos Componentes React (`traducoes_regras_negocio.md`)

**Novo Componente: `ApprovalTimesheetTable.tsx`**
- Uma nova tabela, similar à `TimesheetTable` do colaborador, mas com:
  - Coluna "User".
  - Coluna "Status" com badges coloridos.
  - Coluna "Actions" com os ícones de aprovar, rejeitar e editar.
  - Lógica para filtros avançados (usuário, status, etc.).
  - Lógica para ações em massa.

**Atualização do `TimesheetRouter.tsx`:**
- Deve renderizar a `ApprovalTimesheetTable` se o usuário tiver a role `team_leader`.

### 3.3 Novas Edge Functions do Supabase

1.  **`on-timesheet-approved`:**
    -   Trigger: `UPDATE` na `timesheet_entries` onde `status` muda para `approved`.
    -   Ação: Cria Pedido de Compra (se PJ), envia e-mail de notificação.

2.  **`on-timesheet-rejected`:**
    -   Trigger: `UPDATE` na `timesheet_entries` onde `status` muda para `rejected`.
    -   Ação: Envia e-mail para o colaborador com o `rejection_reason`.

3.  **`on-timesheet-submitted`:**
    -   Trigger: `UPDATE` na `timesheet_entries` onde `status` muda para `pending`.
    -   Ação: Envia e-mail para o Team Leader daquele usuário.

---

## 4. Checklist de Implementação (Visão do Gestor)

1.  **Backend:**
    -   [ ] Aplicar as novas migrations no Supabase (novas tabelas e campos).
    -   [ ] Criar as 3 Edge Functions para automação de e-mails e pedidos.
    -   [ ] Implementar a RLS (Row-Level Security) para que gestores só vejam suas equipes.
    -   [ ] Criar o trigger de auditoria para a tabela `timesheet_edits`.

2.  **Frontend:**
    -   [ ] Desenvolver o componente `ApprovalTimesheetTable.tsx`.
    -   [ ] Implementar os filtros avançados (status, usuário, etc.).
    -   [ ] Criar o modal de rejeição com campo de motivo obrigatório.
    -   [ ] Integrar as chamadas de API para aprovar, rejeitar e editar.
    -   [ ] Adicionar os novos itens de menu ("All times", "Reporting") na sidebar para o perfil de gestor.

---

## 5. Conclusão

Com a adição do fluxo do Team Leader, a arquitetura do módulo de timesheet está completa. A solução cobre desde o apontamento do colaborador até a aprovação final do gestor, com automações importantes para e-mails e criação de pedidos de compra, garantindo um processo robusto, auditável e eficiente para o Humanamente.
