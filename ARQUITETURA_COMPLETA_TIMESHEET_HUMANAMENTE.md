# Arquitetura Completa: Módulo de Timesheet e Aprovações - Humanamente

**Projeto:** Humanamente - Substituição do Módulo de Horas e Ponto  
**Baseado em:** Análise Interativa das Demos Kimai e MarcaPonto  
**Autor:** Manus AI  
**Data:** 01 de Fevereiro de 2026

---

## 🎯 SUMÁRIO EXECUTIVO

Este documento consolida **toda a arquitetura visual, funcional e técnica** para o novo módulo de apontamento e aprovação de horas do Humanamente. Ele unifica as análises dos perfis **Colaborador (PJ e CLT)** e **Team Leader (Gestor)**, fornecendo um guia único e completo para a implementação.

### Visão Geral da Solução

A solução proposta abandona a integração direta com sistemas legados em favor de uma **tradução nativa** dos melhores conceitos para a stack do Humanamente (React, TypeScript, Supabase, Tailwind CSS). Isso garante performance, manutenibilidade e uma experiência de usuário coesa.

**Diferenciação de Perfis:**
-   **Colaborador PJ:** Interface rica inspirada no **Kimai**, focada em apontamento por projeto, timer e valor/hora.
-   **Colaborador CLT:** Interface simplificada inspirada no **MarcaPonto**, focada em batida de ponto com geolocalização.
-   **Team Leader (Gestor):** Interface de gestão inspirada no **Kimai**, com visão da equipe, filtros avançados e fluxo de aprovação.

### Fluxo End-to-End

1.  **Apontamento:** Colaboradores (PJ ou CLT) registram suas horas.
2.  **Submissão:** Ao final do período, submetem os registros para aprovação (status muda para `pending`).
3.  **Notificação:** Gestor é notificado por e-mail sobre as horas pendentes.
4.  **Aprovação:** Gestor acessa a tela de aprovação, analisa e aprova ou rejeita os registros.
5.  **Automação:**
    -   **Se Aprovado (PJ):** Um Pedido de Compra é criado automaticamente.
    -   **Se Rejeitado:** Colaborador é notificado com o motivo.
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

### 1.2 Wireframe: Interface do Colaborador PJ (Baseado no Kimai)

**Tela Principal: "Meus Horários"**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER: [▶️ Timer: 0:00] [🔄 Reiniciar] [👤 Avatar: João Silva (PJ)]         │
├─────────────────────────────────────────────────────────────────────────────┤
│ SIDEBAR │ Meus horários                                                     │
│         ├───────────────────────────────────────────────────────────────────┤
│ ☰ Menu  │ [🔲 Colunas] [🔽 Período] [🔍 Filtros] [Procurar...]    [+ Criar] [⬇ Exportar]│
│         ├───────────────────────────────────────────────────────────────────┤
│         │ ┌─┬────┬───────┬─────────┬─────────┬─────────┬─────────┬─────────┐ │
│         │ │✅│Data│Projeto│Atividade│Duração  │Valor    │Status   │Ações    │ │
│         │ ├─┼────┼───────┼─────────┼─────────┼─────────┼─────────┼─────────┤ │
│         │ │✅│... │ ...   │ ...     │ 8:00    │ R$800   │ Rascunho│▶ ✏️ 🗑️ │ │
│         │ └─┴────┴───────┴─────────┴─────────┴─────────┴─────────┴─────────┘ │
│         │ ┌───────────────────────────────────────────────────────────────┐ │
│         │ │ Ações em Massa: [Submeter para Aprovação] [Exportar] [Deletar]│ │
│         │ └───────────────────────────────────────────────────────────────┘ │
└─────────┴───────────────────────────────────────────────────────────────────┘
```

**Modal de Criação (PJ):**
```
┌───────────────────────────────────────────────────────────┐
│ Criar Registro de Tempo                            ? ×    │
├───────────────────────────────────────────────────────────┤
│ De: [📅 Data] [🕐 Hora]   Duração/Fim: [⏱ Duração] [🕐 Fim] │
│ Projeto*: [🔽 Selecione um projeto...]                      │
│ Atividade*: [🔽 Selecione uma atividade...]                 │
│ Descrição: [Caixa de texto para detalhes...]              │
│ Faturável? [✅]   Valor/Hora: [R$ 100,00]                   │
│                                    [Salvar] [Fechar]      │
└───────────────────────────────────────────────────────────┘
```

### 1.3 Wireframe: Interface do Colaborador CLT (Baseado no MarcaPonto)

**Tela Principal: "Bater Ponto"**
```
┌──────────────────────────────────────────────────────────────────┐
│ HEADER: [👤 Avatar: Maria Santos (CLT)]                          │
├──────────────────────────────────────────────────────────────────┤
│ SIDEBAR │ Bater Ponto                                            │
│         ├────────────────────────────────────────────────────────┤
│ ☰ Menu  │ ┌────────────────────────────────────────────────────┐ │
│         │ │                  1 de Fevereiro de 2026              │ │
│         │ │                       14:30:55                       │ │
│         │ │                                                    │ │
│         │ │            [🟢 BATER PONTO]                        │ │
│         │ │                                                    │ │
│         │ │ 📍 Localização: OK (Rua Exemplo, 123)              │ │
│         │ └────────────────────────────────────────────────────┘ │
│         │ ┌────────────────────────────────────────────────────┐ │
│         │ │ Registros de Hoje:                                 │ │
│         │ │  - Entrada: 09:01:15                               │ │
│         │ │  - Saída (Almoço): 12:30:05                        │ │
│         │ └────────────────────────────────────────────────────┘ │
└─────────┴────────────────────────────────────────────────────────┘
```

### 1.4 Wireframe: Interface do Team Leader (Aprovação)

**Tela Principal: "Aprovação de Horas"**
```
┌───────────────────────────────────────────────────────────────────────────────────────────┐
│ HEADER: [👤 Avatar: Tony Maier (Gestor)]                                                  │
├───────────────────────────────────────────────────────────────────────────────────────────┤
│ SIDEBAR │ Aprovação de Horas                                                              │
│         ├─────────────────────────────────────────────────────────────────────────────────┤
│ ☰ Menu  │ [Período▼] [Status: Pendente▼] [Usuário▼] [Tipo▼]  🔍    [+ Criar] [Exportar▼]   │
│         ├─────────────────────────────────────────────────────────────────────────────────┤
│         │ ☑️ │ Usuário      │ Data     │ Projeto │ Horas │ Tipo │ Valor   │ Status   │ Ações │
│         │───┼──────────────┼──────────┼─────────┼───────┼──────┼─────────┼──────────┼───────│
│         │ ☑ │ João Silva   │ 01/02/26 │ Proj A  │ 8:00  │ PJ   │ R$800   │ Pendente │ ✓ ✗ ✎ │
│         │ ☑ │ Maria Santos │ 01/02/26 │ -       │ 8:02  │ CLT  │ —       │ Pendente │ ✓ ✗ ✎ │
│         │ ☑ │ Pedro Costa  │ 31/01/26 │ Proj A  │ 10:00 │ PJ   │ R$1.200 │ Pendente │ ✓ ✗ ✎ │
│         ├─────────────────────────────────────────────────────────────────────────────────┤
│         │ Ações em Massa: [Aprovar Selecionados] [Rejeitar Selecionados]                  │
└─────────┴─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. ARQUITETURA TÉCNICA

### 2.1 Estrutura do Banco de Dados (Supabase - PostgreSQL)

**Tabela `profiles` (existente):**
- Adicionar campo `contract_type` (enum: `pj`, `clt`).
- Adicionar campo `hourly_rate` (numeric, nullable).
- Adicionar campo `team_id` (uuid, FK para `teams`), para hierarquia.

**Tabela `timesheet_entries` (principal):**
```sql
CREATE TABLE timesheet_entries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) NOT NULL,
  project_id UUID REFERENCES projects(id), -- Nulo para CLT
  activity_id UUID REFERENCES activities(id), -- Nulo para CLT
  start_time TIMESTAMPTZ NOT NULL,
  end_time TIMESTAMPTZ,
  duration INT, -- Em segundos
  description TEXT,
  is_billable BOOLEAN DEFAULT true,
  status TEXT NOT NULL DEFAULT 'draft', -- draft, pending, approved, rejected
  rejection_reason TEXT,
  approved_by UUID REFERENCES profiles(id),
  approved_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

**Tabela `clock_entries` (para CLT):**
```sql
CREATE TABLE clock_entries (
  id BIGSERIAL PRIMARY KEY,
  user_id UUID REFERENCES profiles(id) NOT NULL,
  clock_in_time TIMESTAMPTZ NOT NULL,
  latitude NUMERIC(10, 7),
  longitude NUMERIC(10, 7),
  is_within_geofence BOOLEAN
);
```

**Tabela `purchase_orders` (para PJ):**
```sql
CREATE TABLE purchase_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) NOT NULL,
  total_amount NUMERIC(10, 2) NOT NULL,
  period_start DATE NOT NULL,
  period_end DATE NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending_payment', -- pending_payment, paid
  created_at TIMESTAMPTZ DEFAULT now(),
  invoice_requested_at TIMESTAMPTZ
);
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
```

### 2.2 Componentes React Principais

1.  **`TimesheetRouter.tsx`:** Componente de alto nível que verifica o `contract_type` do usuário logado e renderiza a interface correta (PJ ou CLT).
2.  **`KimaiTimesheet.tsx`:** A interface completa para o colaborador PJ, contendo a tabela, filtros e o modal de criação.
3.  **`MarcaPontoClock.tsx`:** A interface simplificada para o colaborador CLT, com o botão de bater ponto e a lista de registros do dia.
4.  **`ApprovalDashboard.tsx`:** A interface do Team Leader, com a tabela de aprovação, filtros avançados e lógica para ações em massa.

### 2.3 Automações (Supabase Edge Functions)

1.  **`on-timesheet-submitted`:**
    -   **Trigger:** `UPDATE` na `timesheet_entries` com `status` mudando para `pending`.
    -   **Ação:** Envia e-mail para o gestor da equipe daquele usuário.

2.  **`on-timesheet-approved`:**
    -   **Trigger:** `UPDATE` na `timesheet_entries` com `status` mudando para `approved`.
    -   **Ação:**
        -   Se `contract_type` for `pj`, cria um registro na `purchase_orders`.
        -   Envia e-mail de notificação para o colaborador.

3.  **`on-timesheet-rejected`:**
    -   **Trigger:** `UPDATE` na `timesheet_entries` com `status` mudando para `rejected`.
    -   **Ação:** Envia e-mail para o colaborador com o `rejection_reason`.

4.  **`on-clock-in` (CLT):**
    -   **Trigger:** `INSERT` na `clock_entries`.
    -   **Ação:** Usa PostGIS para verificar se as coordenadas (`latitude`, `longitude`) estão dentro da área geográfica permitida (`geofence`) e atualiza o campo `is_within_geofence`.

---

## 3. FLUXO DE DADOS E REGRAS DE NEGÓCIO

### 3.1 Submissão para Aprovação
- O colaborador (PJ ou CLT) só pode submeter registros com status `draft`.
- Ao submeter, o status de todos os registros selecionados muda para `pending` e eles se tornam não editáveis para o colaborador.

### 3.2 Aprovação e Rejeição
- Apenas usuários com role `team_leader` ou `admin` podem aprovar/rejeitar.
- Um gestor só pode ver e aprovar registros dos membros da sua equipe (implementado via RLS no Supabase).
- **Rejeição:** O campo `rejection_reason` é obrigatório.
- **Edição pelo Gestor:** Qualquer edição feita por um gestor em um registro de um colaborador deve ser registrada na tabela de auditoria `timesheet_edits`.

### 3.3 Geração de Pedido de Compra (PJ)
- A criação do Pedido de Compra é um processo automatizado que ocorre **após** a aprovação das horas.
- A Edge Function deve consolidar todas as horas aprovadas em um determinado período para gerar um único pedido.

### 3.4 Geolocalização (CLT)
- O frontend deve capturar a geolocalização do navegador no momento da batida de ponto.
- O backend valida se as coordenadas estão dentro de uma área pré-definida (geofence) usando funções do PostGIS.

---

## 4. CHECKLIST DE IMPLEMENTAÇÃO

**Fase 1: Backend (Supabase)**
1.  [ ] Aplicar todas as migrations do banco de dados (novas tabelas e campos).
2.  [ ] Configurar a extensão PostGIS no Supabase.
3.  [ ] Implementar a RLS (Row-Level Security) para a hierarquia de equipes.
4.  [ ] Criar o trigger de auditoria para a tabela `timesheet_edits`.
5.  [ ] Desenvolver e implantar as 4 Edge Functions (submitted, approved, rejected, on-clock-in).

**Fase 2: Frontend (React)**
1.  [ ] Desenvolver o componente `TimesheetRouter.tsx`.
2.  [ ] Desenvolver a interface PJ (`KimaiTimesheet.tsx`) com tabela, filtros e modal.
3.  [ ] Desenvolver a interface CLT (`MarcaPontoClock.tsx`) com captura de geolocalização.
4.  [ ] Desenvolver a interface do Gestor (`ApprovalDashboard.tsx`) com a tabela de aprovação e ações.
5.  [ ] Integrar todas as chamadas de API do Supabase.

**Fase 3: Integrações e Testes**
1.  [ ] Configurar serviço de e-mail (Resend, SendGrid) e integrá-lo às Edge Functions.
2.  [ ] Testar o fluxo end-to-end completo para um colaborador PJ.
3.  [ ] Testar o fluxo end-to-end completo para um colaborador CLT.
4.  [ ] Validar todas as regras de negócio (permissões, validações, automações).
5.  [ ] Realizar testes de segurança para garantir que um usuário não possa ver ou aprovar dados de outra equipe.

---

## 5. CONCLUSÃO

Esta arquitetura unificada fornece um plano de ação claro e detalhado para a equipe de desenvolvimento. Ao seguir este documento, o Humanamente terá um módulo de timesheet robusto, moderno e adaptado às necessidades específicas de cada tipo de colaborador e gestor, com automações que garantem eficiência e auditabilidade.
