# Arquitetura Ajustada: Módulo de Timesheet PJ - Humanamente

**Projeto:** Humanamente - Adaptação do Módulo de Timesheet PJ  
**Baseado em:** Análise do Código Existente e da Demo Kimai  
**Perfil:** Colaboradores PJ (Pessoa Jurídica) e Gestores  
**Autor:** Manus AI  
**Data:** 01 de Fevereiro de 2026

---

## 🎯 SUMÁRIO EXECUTIVO

Este documento detalha a **arquitetura ajustada** para o módulo de apontamento e aprovação de horas do Humanamente, focada em **colaboradores PJ**. A análise do código-fonte existente revelou que uma base sólida já está implementada, incluindo tabelas, componentes e hooks. Portanto, o plano agora é **aprimorar e adaptar** o sistema atual para refletir a experiência de usuário e as funcionalidades do **Kimai**, em vez de criar tudo do zero.

### Diagnóstico da Realidade

-   **Backend:** Já existem tabelas como `timesheet_entries`, `timesheet_projects` e `contractor_profiles`. No entanto, a estrutura é diferente da idealizada (baseada no Kimai). A tabela `contractor_timesheet_entries` parece ser a mais adequada, mas precisa de ajustes.
-   **Frontend:** Componentes como `TimesheetRouter.tsx`, `TimesheetPJ.tsx` e `AprovacaoHoras.tsx` já existem, mas a interface do `TimesheetPJ` é mais simples que a do Kimai e a de aprovação é mais complexa, incluindo abas para Horas Extras e Laudos.
-   **Fluxo:** O roteamento entre PJ e CLT já existe, mas o foco será exclusivamente no aprimoramento do fluxo PJ.

### Estratégia de Adaptação

1.  **Unificar Tabelas:** Migrar a lógica para usar uma única tabela principal de apontamentos (`contractor_timesheet_entries`), ajustando-a para incluir os campos necessários do Kimai (descrição, faturável, etc.).
2.  **Aprimorar a Interface PJ:** Substituir a interface atual do `TimesheetPJ.tsx` por uma que espelhe a do Kimai, com a tabela rica em filtros, o modal de criação completo e o dashboard com gráficos.
3.  **Simplificar a Aprovação:** Adaptar a tela `AprovacaoHoras.tsx` para focar apenas na aprovação de timesheet PJ, removendo as abas de Horas Extras e Laudos (ou mantendo-as se forem relevantes para o fluxo PJ).
4.  **Implementar Funcionalidades Faltantes:** Adicionar o timer ativo no header, a visualização em calendário e as automações via Edge Functions.

---

## 1. ARQUITETURA VISUAL E WIREFRAMES (Kimai)

*(Os wireframes propostos na análise anterior, baseados no Kimai, continuam sendo o **objetivo final** da interface. A implementação atual será adaptada para alcançá-los.)*

### 1.1 Interface do Colaborador PJ (Objetivo)

**Tela Principal: "Meus Horários"**
-   **Tabela Rica:** Com filtros por período, projeto, atividade e status.
-   **Ações em Massa:** Checkboxes para selecionar múltiplos registros e submeter para aprovação.
-   **Modal Completo:** Para criação/edição de registros, com campos para descrição, valor/hora, faturável, etc.
-   **Timer Ativo:** Widget no header global.

**Dashboard:**
-   Gráficos de horas e valores (hoje, semana, mês, ano).
-   Toggle para mostrar/ocultar valores financeiros.

**Calendário:**
-   Visualização mensal/semanal/diária com eventos coloridos por projeto.

### 1.2 Interface do Gestor (Objetivo)

**Tela "Aprovação de Horas":**
-   **Tabela Consolidada:** Visão de todos os registros pendentes da equipe.
-   **Filtros Avançados:** Por usuário, status, período, projeto.
-   **Ações em Massa:** Aprovar/rejeitar múltiplos registros.
-   **Modal de Rejeição:** Com campo de motivo obrigatório.

---

## 2. ARQUITETURA TÉCNICA (AJUSTADA)

### 2.1 Estrutura do Banco de Dados (Adaptação)

**Tabela Principal: `contractor_timesheet_entries` (Ajustar)**
-   **Manter:** `id`, `tenant_id`, `contractor_id`, `project_id`, `clock_in`, `clock_out`, `hourly_rate`, `total_hours`, `status`, `approved_by`, `approved_at`, `rejection_reason`.
-   **Adicionar:**
    -   `activity_id UUID REFERENCES activities(id)`
    -   `description TEXT` (já existe, mas usar para descrição detalhada)
    -   `is_billable BOOLEAN DEFAULT true`
    -   `start_time TIMESTAMPTZ` (pode ser o mesmo que `clock_in`)
    -   `end_time TIMESTAMPTZ` (pode ser o mesmo que `clock_out`)
    -   `duration INT` (em segundos, para maior precisão)
-   **Remover/Substituir:** A lógica de geolocalização (`clock_in_latitude`, etc.) e `device_info` parece mais voltada para batida de ponto CLT e pode ser removida se não for requisito para PJ.

**Tabela `timesheet_projects` (Usar `outsourced_projects`?)**
-   O sistema parece ter duas tabelas de projeto: `timesheet_projects` e `outsourced_projects`. É crucial **unificar** em uma única tabela. A `outsourced_projects` parece mais simples e pode ser a base.

**Tabela `activities` (Criar se não existir)**
-   É fundamental criar uma tabela de atividades vinculada aos projetos para permitir o nível de detalhe do Kimai.

**Tabela `purchase_orders` (Criar)**
-   A tabela para gerar os Pedidos de Compra automaticamente após a aprovação não existe e precisa ser criada.

**Tabela `timesheet_edits` (Criar)**
-   A tabela de auditoria para registrar edições feitas por gestores não existe e precisa ser criada, junto com o trigger.

### 2.2 Componentes React (Adaptação)

**1. `TimesheetPJ.tsx` (Refatorar)**
-   **Substituir a estrutura atual** de abas ("Horas" e "Laudos") pela interface focada em timesheet do Kimai.
-   **Remover a `DeliveredReportsList`** (ou movê-la para outra página) para dar espaço ao dashboard com gráficos e à tabela rica em filtros.
-   **Integrar o `TimesheetTable.tsx`** (que já existe) e aprimorá-lo com filtros e ações em massa.
-   **Substituir o `NewEntryDialog.tsx`** pelo modal mais completo do Kimai.

**2. `AprovacaoHoras.tsx` (Simplificar/Adaptar)**
-   **Focar a aba "Timesheet"** na aprovação dos registros da tabela `contractor_timesheet_entries`.
-   **O `TimesheetApprovalPanel.tsx`** deve ser adaptado para exibir os dados da forma correta e permitir as ações de aprovação/rejeição.
-   As abas "Horas Adicionais", "Laudos", "Por Projeto" e "Por Departamento" devem ser avaliadas. Se não fizerem parte do fluxo PJ, podem ser removidas desta tela para simplificar a interface do gestor.

**3. `TimerWidget.tsx` (Criar)**
-   O componente de timer ativo no header não existe e precisa ser criado e integrado ao layout principal.

**4. `TimesheetCalendar.tsx` (Criar)**
-   A visualização em calendário não existe e precisa ser criada, utilizando bibliotecas como `react-big-calendar`.

### 2.3 Automações (Supabase Edge Functions - Criar)

As Edge Functions para automação de e-mails e criação de Pedidos de Compra não existem e precisam ser criadas do zero, conforme a lógica detalhada na análise anterior.

-   `on-timesheet-submitted`
-   `on-timesheet-approved`
-   `on-timesheet-rejected`

---

## 3. PLANO DE IMPLEMENTAÇÃO (ADAPTADO)

### Fase 1: Backend - Unificação e Criação

-   [ ] **Migration 1: Ajustar `contractor_timesheet_entries`**
    -   Adicionar colunas: `activity_id`, `is_billable`, `duration`.
    -   Remover colunas de geolocalização se não aplicável.
-   [ ] **Migration 2: Unificar Projetos**
    -   Decidir entre `timesheet_projects` e `outsourced_projects`.
    -   Migrar dados se necessário e remover a tabela duplicada.
-   [ ] **Migration 3: Criar `activities`**
    -   Criar a tabela de atividades vinculada aos projetos.
-   [ ] **Migration 4: Criar `purchase_orders` e `timesheet_edits`**
    -   Criar as tabelas de Pedidos de Compra e Auditoria.
-   [ ] **Trigger:** Implementar o trigger de auditoria na tabela `timesheet_edits`.
-   [ ] **Edge Functions:** Desenvolver e testar as 3 Edge Functions para notificações e automações.

### Fase 2: Frontend - Refatoração e Criação

-   [ ] **Refatorar `TimesheetPJ.tsx`:**
    -   Remover a estrutura de abas.
    -   Adicionar o dashboard com gráficos (usar `recharts` ou similar).
    -   Aprimorar `TimesheetTable.tsx` com filtros e ações.
    -   Substituir `NewEntryDialog.tsx`.
-   [ ] **Criar `TimerWidget.tsx`:**
    -   Implementar o timer no header global.
-   [ ] **Criar `TimesheetCalendar.tsx`:**
    -   Implementar a visualização em calendário.
-   [ ] **Adaptar `AprovacaoHoras.tsx`:**
    -   Simplificar a interface, focando na aprovação de timesheet PJ.
    -   Garantir que `TimesheetApprovalPanel.tsx` funcione com a nova estrutura de dados.

### Fase 3: Integrações e Testes

-   [ ] **Conectar Frontend e Backend:** Garantir que todos os componentes estejam usando as tabelas e Edge Functions corretas.
-   [ ] **Testar Fluxo End-to-End:** Simular o processo completo: apontamento → submissão → aprovação → criação de Pedido de Compra.
-   [ ] **Validar Regras de Negócio:** Testar permissões (RLS), validações de formulário e cálculos automáticos.
-   [ ] **Testar Notificações:** Verificar o envio de e-mails em cada etapa do fluxo.

---

## 4. CONCLUSÃO

A base do módulo de timesheet já existe no Humanamente, o que é um ótimo ponto de partida. O foco agora é **refatorar e aprimorar** o sistema atual para alcançar a rica experiência de usuário do Kimai. Esta abordagem é mais eficiente do que uma reescrita completa, pois aproveita o código e a estrutura existentes, garantindo uma integração mais suave com o restante do sistema.
