# Análise Detalhada: Dashboard do Team Leader - Kimai

**Usuário:** Tony Maier (tony_teamlead)  
**Cargo:** Head of Sales (Team Leader)  
**URL:** https://demo-empty.kimai.org/en/dashboard/

---

## 1. DIFERENÇAS ESTRUTURAIS EM RELAÇÃO AO COLABORADOR

### 1.1 Sidebar - Menu Expandido

O Team Leader tem acesso a **funcionalidades adicionais** que não aparecem para colaboradores comuns:

**Menu Completo (Team Leader):**
- Dashboard
- **Time Tracking (expansível):**
  - My times
  - Weekly hours
  - Calendar
  - **Export** ⭐ (novo)
  - **All times** ⭐ (novo - visão de toda a equipe)
- Employment contract
- Reporting
- **Invoices** ⭐ (novo - faturamento)
- Expenses
- Tasks
- **Administration** ⭐ (novo - configurações do sistema)

**Funcionalidades Exclusivas do Gestor:**
1. **Export:** Permite exportar dados de toda a equipe
2. **All times:** Tela para visualizar e aprovar horas de todos os colaboradores
3. **Invoices:** Gestão de faturas e cobranças
4. **Administration:** Configurações de projetos, usuários, clientes, etc.

---

## 2. DASHBOARD DO TEAM LEADER

### 2.1 Estrutura Visual

O Dashboard do Team Leader é **idêntico visualmente** ao do colaborador, mas com uma diferença crucial: ele pode ver **dados de toda a equipe** se configurado.

**Layout:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER GLOBAL (Timer: 0:00 | Reiniciar | Avatar: Tony Maier - Head of Sales)│
├─────────────────────────────────────────────────────────────────────────────┤
│ SIDEBAR │ Dashboard                                                         │
│         ├───────────────────────────────────────────────────────────────────┤
│ ☰ Menu  │ ┌───────────────────────────────────────────────────────────────┐ │
│         │ │ My working hours                        [<] [Week 05] [>]     │ │
│ • Dash  │ │ ┌───────────────────────────────────────────────────────────┐ │ │
│ • Time  │ │ │ [Gráfico de Linha: Horas por Dia na Semana]               │ │ │
│   - My  │ │ └───────────────────────────────────────────────────────────┘ │ │
│   - Wee │ │ ┌───────┬───────┬───────┬───────┐                            │ │
│   - Cal │ │ │ 0:00  │ 0:00  │ 0:00  │ 0:00  │                            │ │
│   - Exp │ │ │ Today │ Week  │ Jan   │ 2026  │                            │ │
│   - All │ │ └───────┴───────┴───────┴───────┘                            │ │
│ • Empl  │ └───────────────────────────────────────────────────────────────┘ │
│ • Repo  │ ┌──────────────────────┬──────────────────────┐                  │
│ • Invo  │ │ My tasks        [1]  │ Pending tasks   [1]  │                  │
│ • Expe  │ └──────────────────────┴──────────────────────┘                  │
│ • Task  │ ┌──────────────────────┬──────────────────────┐                  │
│ • Admi  │ │ Expenses today  0.00 │ Expenses week   0.00 │                  │
│         │ │ Expenses month  0.00 │ Expenses year   0.00 │                  │
│         │ └──────────────────────┴──────────────────────┘                  │
└─────────┴───────────────────────────────────────────────────────────────────┘
```

### 2.2 Widgets do Dashboard

**Widget 1: "My working hours"**
- Gráfico de linha mostrando horas trabalhadas por dia
- Navegação por semana (setas < >)
- 4 cards de resumo: Hoje, Semana, Mês, Ano

**Widget 2: "My tasks" e "Pending tasks"**
- Contadores de tarefas pessoais e pendentes
- Clicáveis para acessar a lista completa

**Widget 3: "Expenses"**
- Resumo de despesas por período (hoje, semana, mês, ano)
- Valores em moeda configurada

**Observação Importante:**
O Dashboard mostra apenas os dados **pessoais** do Team Leader. Para ver dados da equipe, ele precisa acessar a tela **"All times"**.

---

## 3. HEADER GLOBAL - DIFERENÇAS

**Informações Exibidas:**
- **Timer:** 0:00 (mesmo comportamento do colaborador)
- **Botão Reiniciar:** Ícone de histórico
- **Avatar e Nome:** "Tony Maier"
- **Cargo:** "Head of Sales" (exibido abaixo do nome)
- **Menu Pessoal:** Dropdown com "Settings" e "Logout"

**Diferença Visual:**
O cargo ("Head of Sales") aparece abaixo do nome, indicando visualmente que este usuário tem permissões especiais.

---

## 4. PRÓXIMOS PASSOS

Para documentar o fluxo completo de aprovação, preciso acessar:
1. **"All times"** - Tela principal de visualização e aprovação de horas da equipe
2. **"Reporting"** - Relatórios consolidados
3. **"Administration"** - Configurações de projetos e usuários (se relevante)

A tela **"All times"** é onde o fluxo de aprovação acontece.
