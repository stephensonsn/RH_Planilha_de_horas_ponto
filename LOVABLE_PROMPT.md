'''
# Prompt Otimizado para Implementação no Lovable

**COLE O CONTEÚDO ABAIXO DIRETAMENTE NO LOVABLE**

---

## Persona

Assuma a persona de um **Arquiteto de Software Full-Stack Sênior**, especialista em **React, TypeScript, Tailwind CSS e Supabase**. Você é pragmático, focado em criar código limpo, escalável e de alta qualidade, seguindo as melhores práticas do mercado.

## Objetivo Principal

Implementar um **módulo completo de apontamento de horas para colaboradores PJ** no projeto **Humanamente**, baseado na funcionalidade e no design do software de referência **Kimai**. O sistema deve ser robusto, intuitivo e perfeitamente integrado à estrutura existente do Humanamente.

## Contexto do Projeto

- **Projeto:** Humanamente
- **Repositório:** https://github.com/stephensonsn/humanaMente
- **Stack:** React, TypeScript, Next.js, Tailwind CSS, shadcn/ui, Supabase
- **Referência Visual e Funcional:** Kimai (https://demo-empty.kimai.org)
- **Base de Código Existente:** O projeto já possui uma estrutura com autenticação, perfis de usuário e um módulo de batida de ponto para CLT. Sua tarefa é **aprimorar e expandir** o que já existe para o fluxo PJ.

## Requisitos Detalhados

### Fase 1: Backend (Supabase)

#### 1.1. Migrations do Banco de Dados

Crie e aplique as seguintes migrations no Supabase para ajustar e complementar o schema existente:

- **Tabela `projects` (ou `timesheet_projects`):**
  - Garanta que exista uma tabela unificada para projetos.
  - Adicione um campo `hourly_rate` (numeric) para definir o valor/hora padrão do projeto.

- **Tabela `activities`:**
  - Crie uma tabela para atividades vinculadas a projetos.
  - Campos: `id` (uuid), `project_id` (fk), `name` (text), `created_at`.

- **Tabela `contractor_timesheet_entries`:**
  - **Ajuste** a tabela existente para incluir os seguintes campos:
    - `activity_id` (fk para `activities.id`)
    - `duration` (integer, em segundos)
    - `is_billable` (boolean, default: true)
    - `status` (text, com valores: `pending`, `submitted`, `approved`, `rejected`, default: `pending`)

- **Tabela `purchase_orders`:**
  - Crie uma tabela para armazenar os pedidos de compra gerados automaticamente.
  - Campos: `id` (uuid), `contractor_id` (fk), `total_hours` (numeric), `total_amount` (numeric), `period_start` (date), `period_end` (date), `status` (text, default: `pending_payment`), `created_at`.

- **Tabela `timesheet_edits`:**
  - Crie uma tabela de auditoria para registrar edições em apontamentos aprovados.
  - Campos: `id` (uuid), `entry_id` (fk), `editor_id` (fk), `previous_data` (jsonb), `new_data` (jsonb), `reason` (text), `edited_at`.

#### 1.2. Edge Functions

Implemente 3 Edge Functions para automação:

1.  **`on-timesheet-submitted`:**
    - **Trigger:** Quando o status de um ou mais `contractor_timesheet_entries` muda para `submitted`.
    - **Ação:** Envia um e-mail para o gestor da equipe notificando sobre a nova submissão.

2.  **`on-timesheet-approved`:**
    - **Trigger:** Quando o status de um ou mais `contractor_timesheet_entries` muda para `approved`.
    - **Ação:**
        - Cria um registro na tabela `purchase_orders` com o valor total das horas aprovadas.
        - Envia um e-mail para o colaborador e para o financeiro (`financeiro@imepcorp.com.br`) notificando sobre a aprovação e o pedido de compra gerado.

3.  **`on-timesheet-rejected`:**
    - **Trigger:** Quando o status de um ou mais `contractor_timesheet_entries` muda para `rejected`.
    - **Ação:** Envia um e-mail para o colaborador informando sobre a rejeição e incluindo o motivo.

#### 1.3. Row-Level Security (RLS)

Configure as políticas de RLS para garantir que:
- Colaboradores PJ só possam ver e editar seus próprios apontamentos.
- Gestores (Team Leaders) só possam ver e aprovar os apontamentos dos membros de sua equipe.

---

### Fase 2: Frontend (React + TypeScript + shadcn/ui)

Implemente os seguintes componentes, seguindo a estrutura de arquivos e o design do Kimai.

#### 2.1. Componentes Principais

1.  **`TimesheetPJDashboard.tsx`:**
    - Crie o layout principal com abas (`Tabs` da shadcn/ui) para "Meus Horários" e "Calendário".
    - Integre o `TimerWidget` no header.

2.  **`TimerWidget.tsx`:**
    - Componente de timer ativo no header.
    - Use **Zustand** para gerenciamento de estado global.
    - Funcionalidades: Play/Pause, Stop (que salva o registro), seleção de projeto/atividade.

3.  **`TimesheetTable.tsx`:**
    - Tabela (`Table` da shadcn/ui) para listar os registros.
    - Use **React Query** para buscar e fazer cache dos dados.
    - Implemente filtros por período, projeto e status (`Select` e `DatePicker` da shadcn/ui).
    - Implemente ações em massa (seleção com `Checkbox`) e por linha (`DropdownMenu`).

4.  **`TimesheetModal.tsx`:**
    - Modal (`Dialog` da shadcn/ui) para criar/editar registros.
    - Use **React Hook Form** e **Zod** para validação.
    - Use `Combobox` (Command + Popover) para seleção com busca de projetos e atividades.
    - Calcule e exiba a duração e o valor automaticamente.

5.  **`TimesheetCalendar.tsx`:**
    - Integre a biblioteca **`react-big-calendar`**.
    - Exiba os registros em visualizações de mês, semana e dia.
    - Estilize o calendário para combinar com o tema do Humanamente.

6.  **`ApprovalDashboard.tsx`:**
    - Crie a interface para o gestor (Team Leader).
    - Reutilize o `TimesheetTable`, mas com colunas adicionais (Colaborador) e ações de Aprovar/Rejeitar.
    - Crie um modal para inserir o motivo da rejeição (obrigatório).

#### 2.2. Hooks e Lógica

- Crie hooks customizados (ex: `useTimesheetQueries.ts`) para encapsular a lógica de `useQuery` e `useMutation`.
- Crie um store com **Zustand** (`timerStore.ts`) para o timer global.

## Saída Esperada

- **Código-fonte completo** dos componentes React e hooks.
- **Arquivos de migration SQL** para o Supabase.
- **Código-fonte das Edge Functions**.
- **Instruções claras** sobre como integrar os novos componentes na estrutura existente do Humanamente.
- O código deve ser limpo, comentado e seguir os padrões do projeto.

---

**Execute esta implementação, garantindo que o resultado final seja uma réplica funcional e visualmente fiel do sistema Kimai, adaptado à realidade e à stack tecnológica do Humanamente.**
'''
