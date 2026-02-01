# Wireframes e Componentes React para Timesheet PJ (Kimai)

**Projeto:** Humanamente - Módulo de Apontamento de Horas PJ  
**Stack:** React + TypeScript + Tailwind CSS + Supabase  
**Referência Visual:** Kimai (https://demo-empty.kimai.org)  
**Data:** 01/02/2026

---

## 1. VISÃO GERAL DA INTERFACE

A interface do colaborador PJ será composta por 3 áreas principais, organizadas em abas para manter a navegação limpa e intuitiva, seguindo o padrão do Kimai.

- **Meus Horários:** Tabela completa para visualização, edição e submissão de registros.
- **Calendário:** Visualização interativa dos registros em formato mensal, semanal ou diário.
- **Relatórios:** (Futuro) Gráficos e resumos de horas trabalhadas.

O **Timer Ativo** será um componente global, sempre visível no header da aplicação, permitindo que o colaborador inicie, pause e pare o cronômetro de qualquer tela.

---

## 2. WIREFRAMES E COMPONENTES

### 2.1. Tela Principal: `TimesheetPJDashboard.tsx`

**Objetivo:** Unificar a experiência do usuário em uma única tela com abas, mantendo o design do Humanamente e a funcionalidade do Kimai.

**Wireframe:**
```
┌───────────────────────────────────────────────────────────────────────────────────┐
│ 👤 Olá, Fabrizio Castro                                       [▶️ Timer: 0:00] [⚙️] │
├───────────────────────────────────────────────────────────────────────────────────┤
│                                                                                   │
│  Planilha de Horas - PJ                                                           │
│  Gerencie seus apontamentos de horas, projetos e atividades.                      │
│                                                                                   │
│  ┌──────────────────┬──────────────────┬──────────────────┐                       │
│  │ 📋 Meus Horários │  📅 Calendário   │  📈 Relatórios   │                       │
│  └──────────────────┴──────────────────┴──────────────────┘                       │
│                                                                                   │
│  ┌───────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                               │  │
│  │  (Conteúdo da Aba Ativa: TimesheetTable ou TimesheetCalendar)                 │  │
│  │                                                                               │  │
│  └───────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                   │
└───────────────────────────────────────────────────────────────────────────────────┘
```

**Componentes Utilizados:**
- **Layout:** `div` com Flexbox (Tailwind CSS)
- **Abas:** `Tabs`, `TabsList`, `TabsTrigger`, `TabsContent` (shadcn/ui)
- **Header:** Componente customizado com `TimerWidget`

---

### 2.2. Aba "Meus Horários": `TimesheetTable.tsx`

**Objetivo:** Fornecer uma visão detalhada e funcional de todos os registros de horas, com filtros poderosos e ações rápidas.

**Wireframe:**
```
┌───────────────────────────────────────────────────────────────────────────────────┐
│ [📅 Período: Este Mês ▼] [📁 Projeto: Todos ▼] [🏷️ Status: Pendente ▼] [🔍 Buscar...] │
├───────────────────────────────────────────────────────────────────────────────────┤
│                                                                                   │
│  ┌───────────────────────────────────────────────────────────────────────────────┐  │
│  │ ☑️ │ Data       │ Projeto    │ Atividade  │ Duração │ Valor  │ Status   │ Ações │  │
│  ├────┼────────────┼────────────┼────────────┼─────────┼────────┼──────────┼───────┤  │
│  │ ☑️ │ 01/02/2026 │ Website    │ Frontend   │ 2h 30m  │ R$ 250 │ Pendente │  ⋮    │  │
│  │ ☐  │ 31/01/2026 │ App Mobile │ Backend    │ 4h 00m  │ R$ 400 │ Aprovado │  ⋮    │  │
│  │ ☐  │ 30/01/2026 │ Website    │ Design     │ 1h 15m  │ R$ 125 │ Rejeitado│  ⋮    │  │
│  └───────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                   │
├───────────────────────────────────────────────────────────────────────────────────┤
│  Total Selecionado: 2h 30m (R$ 250,00)        [Submeter Selecionados para Aprovação] │
└───────────────────────────────────────────────────────────────────────────────────┘
```

**Componentes Utilizados:**
- **Tabela:** `Table`, `TableHeader`, `TableBody`, `TableRow`, `TableHead`, `TableCell` (shadcn/ui)
- **Filtros:**
  - `Select` (shadcn/ui) para Período, Projeto e Status
  - `Input` (shadcn/ui) para busca textual
  - `DatePicker with Range` (shadcn/ui) para período customizado
- **Ações:**
  - `Checkbox` (shadcn/ui) para seleção
  - `DropdownMenu` (shadcn/ui) para ações por linha (Editar, Excluir, Duplicar)
  - `Button` (shadcn/ui) para "Submeter Selecionados"
- **Status:** `Badge` (shadcn/ui) com cores diferentes para cada status (Pendente, Aprovado, Rejeitado)

---

### 2.3. Modal de Criação/Edição: `TimesheetModal.tsx`

**Objetivo:** Oferecer um formulário completo e intuitivo para o lançamento de horas, com validação e cálculo automático.

**Wireframe:**
```
┌────────────────────────────────────────────────────────┐
│ ✖️ Novo Registro de Horas                              │
├────────────────────────────────────────────────────────┤
│ Projeto *                                              │
│ [🔍 Selecione o projeto...                      ▼]      │
│                                                        │
│ Atividade *                                            │
│ [🔍 Selecione a atividade...                     ▼]      │
│                                                        │
│ Data *                    │ Hora Início * │ Hora Fim * │
│ [📅 01/02/2026]           │ [09:00]       │ [11:30]    │
│                                                        │
│ Descrição                                              │
│ [Desenvolvimento da tela de login...]                  │
│                                                        │
│ ☑️ Faturável                                           │
│                                                        │
│ ┌────────────────────────────────────────────────────┐ │
│ │ Duração Calculada: 2h 30m │ Valor: R$ 250,00       │ │
│ └────────────────────────────────────────────────────┘ │
├────────────────────────────────────────────────────────┤
│                          [Cancelar] [Salvar]           │
└────────────────────────────────────────────────────────┘
```

**Componentes Utilizados:**
- **Modal:** `Dialog` (shadcn/ui)
- **Formulário:** `Form` (shadcn/ui) com `react-hook-form` e `zod`
- **Seleção com Busca:** `Combobox` (shadcn/ui - `Command` + `Popover`) para Projeto e Atividade
- **Data:** `DatePicker` (shadcn/ui - `Calendar` + `Popover`)
- **Horários:** `Input` com `type="time"`
- **Descrição:** `Textarea` (shadcn/ui)
- **Checkbox:** `Checkbox` (shadcn/ui)
- **Resumo:** `Card` (shadcn/ui) para Duração e Valor

---

### 2.4. Aba "Calendário": `TimesheetCalendar.tsx`

**Objetivo:** Proporcionar uma visão visual e interativa dos registros, facilitando a identificação de dias com ou sem apontamentos.

**Wireframe:**
```
┌───────────────────────────────────────────────────────────────────────────────────┐
│ [◀️] Fevereiro 2026 [▶️]                                [Hoje] [Mês] [Semana] [Dia] │
├───────────────────────────────────────────────────────────────────────────────────┤
│                                                                                   │
│  Dom   │   Seg   │   Ter   │   Qua   │   Qui   │   Sex   │   Sáb                   │
│ ───────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────                │
│   25   │    26   │    27   │    28   │    29   │    30   │    31                   │
│        │         │         │         │         │         │                         │
│ ───────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────                │
│    1   │     2   │     3   │     4   │     5   │     6   │     7                   │
│  2h30m │   4h    │         │         │         │         │                         │
│ ───────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────                │
│    8   │     9   │    10   │    11   │    12   │    13   │    14                   │
│        │         │         │         │         │         │                         │
│                                                                                   │
└───────────────────────────────────────────────────────────────────────────────────┘
```

**Componentes Utilizados:**
- **Biblioteca Principal:** `react-big-calendar`
- **Navegação:** `Button` e `ButtonGroup` (shadcn/ui) para controle de período e visualização
- **Estilização:** CSS customizado para integrar o `react-big-calendar` ao tema do Humanamente (cores, fontes, bordas)

---

## 3. FLUXO DE DADOS E INTERAÇÃO

1. **Carregamento:** Ao entrar na tela, `useQuery` (React Query) busca os registros do Supabase.
2. **Timer:** O `TimerWidget` usa `zustand` para gerenciar o estado do cronômetro globalmente.
3. **Criação/Edição:** O `TimesheetModal` usa `react-hook-form` para gerenciar o formulário e `useMutation` para salvar os dados.
4. **Filtros:** A alteração nos filtros dispara uma nova busca no Supabase via `useQuery`.
5. **Submissão:** A ação de submeter chama uma `Edge Function` no Supabase para alterar o status dos registros e notificar o gestor.

---

## 4. CHECKLIST DE IMPLEMENTAÇÃO

**Fase 1: Estrutura e Componentes Base (2-3 dias)**
- [ ] Criar `TimesheetPJDashboard.tsx` com abas.
- [ ] Implementar `TimesheetTable.tsx` com dados mockados.
- [ ] Implementar `TimesheetModal.tsx` com formulário completo.

**Fase 2: Funcionalidades Avançadas (2-3 dias)**
- [ ] Implementar `TimerWidget.tsx` com store Zustand.
- [ ] Integrar `react-big-calendar` no `TimesheetCalendar.tsx`.
- [ ] Desenvolver filtros avançados para a tabela.

**Fase 3: Integração e Finalização (1-2 dias)**
- [ ] Conectar todos os componentes ao Supabase com React Query.
- [ ] Implementar lógica de submissão e atualização de status.
- [ ] Testar o fluxo completo e ajustar a responsividade.

---

**Próximo Passo:** Entregar esta documentação completa para o usuário.
