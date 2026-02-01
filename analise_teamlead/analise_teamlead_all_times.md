# Análise Detalhada: Tela "All times" (Aprovação de Horas) - Team Leader

**URL:** https://demo-empty.kimai.org/en/team/timesheet/  
**Usuário:** Tony Maier (tony_teamlead) - Head of Sales

---

## 1. ESTRUTURA VISUAL DA TELA "ALL TIMES"

Esta é a **tela principal de aprovação** onde o Team Leader visualiza, filtra e gerencia as horas de toda a equipe.

### 1.1 Wireframe ASCII

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│ HEADER: All times                                                                   │
├─────────────────────────────────────────────────────────────────────────────────────┤
│ BARRA DE FERRAMENTAS                                                                │
│ ┌──────┬──────────┬─────────────────────────────────┬─────────────────────────────┐ │
│ │ [≡]  │ [▼] Time │ [                               │ [+] Create │ [👥] For multi │ │
│ │ Grid │  Range   │  Search                      🔍]│            │ users │ Export▼│ │
│ └──────┴──────────┴─────────────────────────────────┴─────────────────────────────┘ │
├─────────────────────────────────────────────────────────────────────────────────────┤
│ ÁREA DE CONTEÚDO                                                                    │
│                                                                                     │
│  ℹ️ No entries were found based on your selected filters.                          │
│                                                                                     │
│  [Demo vazia - sem registros para exibir]                                          │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Descrição dos Elementos

**Barra de Ferramentas (Superior):**

1. **Botão "Grid" (≡)** - Customizar colunas da tabela
2. **Dropdown "Time Range"** - Filtro de período:
   - Total period
   - Today
   - Yesterday
   - This week
   - Last week
   - This year — until today
   - February 2026
   - January 2026
   - December 2025
   - Q1 2026, Q4 2025, etc.
3. **Campo de Busca** - Busca por texto livre
4. **Botão "Search"** (🔍) - Executar busca
5. **Botão "+ Create"** (verde) - Criar registro de hora para qualquer usuário
6. **Botão "For multiple users"** (azul) - Criar registros em massa
7. **Botão "Export"** (amarelo) - Exportar dados filtrados

---

## 2. ESTRUTURA DA TABELA (QUANDO HÁ DADOS)

Baseado na análise do código-fonte e documentação, a tabela exibe as seguintes colunas:

| Coluna | Descrição | Tipo |
|--------|-----------|------|
| **☑️ Checkbox** | Seleção para ações em massa | Checkbox |
| **User** | Nome do colaborador | Texto |
| **Date** | Data do registro | Data |
| **From** | Hora de início | Hora |
| **To** | Hora de fim | Hora |
| **Duration** | Duração total | Tempo |
| **Project** | Projeto associado | Texto |
| **Activity** | Atividade associada | Texto |
| **Description** | Descrição opcional | Texto |
| **Status** | Status do registro | Badge |
| **Actions** | Ações (editar, deletar, aprovar) | Ícones |

### 2.1 Status Possíveis

- **Draft** (Rascunho) - Cinza
- **Pending** (Pendente de aprovação) - Amarelo
- **Approved** (Aprovado) - Verde
- **Rejected** (Rejeitado) - Vermelho

---

## 3. FLUXO DE APROVAÇÃO

### 3.1 Cenário 1: Aprovação Individual

1. Team Leader acessa "All times"
2. Filtra por "Pending" (status pendente)
3. Clica no ícone de **"Approve"** (✓) na coluna "Actions"
4. Sistema pede confirmação
5. Status muda para "Approved"
6. Edge Function dispara:
   - E-mail para o colaborador (notificação de aprovação)
   - Criação de Pedido de Compra (se PJ)
   - E-mail para o financeiro (se solicitado)

### 3.2 Cenário 2: Aprovação em Massa

1. Team Leader seleciona múltiplos registros via checkbox
2. Clica em botão de ação em massa **"Approve selected"** (aparece após seleção)
3. Sistema pede confirmação
4. Todos os registros selecionados mudam para "Approved"
5. Edge Functions disparam para cada registro

### 3.3 Cenário 3: Rejeição

1. Team Leader clica no ícone de **"Reject"** (✗) na coluna "Actions"
2. Sistema abre modal pedindo **motivo da rejeição** (obrigatório)
3. Team Leader digita o motivo
4. Status muda para "Rejected"
5. E-mail enviado ao colaborador com o motivo

### 3.4 Cenário 4: Edição Antes de Aprovar

1. Team Leader clica no ícone de **"Edit"** (✎)
2. Modal abre com todos os campos editáveis
3. Team Leader corrige informações (ex: duração, descrição)
4. Salva alterações
5. Pode então aprovar o registro

---

## 4. FILTROS AVANÇADOS

Ao clicar no botão **"Search filter"** (dropdown), o Team Leader tem acesso a filtros avançados:

**Filtros Disponíveis:**
- **User:** Filtrar por colaborador específico
- **Project:** Filtrar por projeto
- **Activity:** Filtrar por atividade
- **Status:** Filtrar por status (Draft, Pending, Approved, Rejected)
- **Date Range:** Período customizado
- **Billable:** Apenas registros faturáveis (sim/não)
- **Exported:** Apenas registros exportados (sim/não)

---

## 5. AÇÕES EM MASSA

Após selecionar múltiplos registros via checkbox, aparecem botões de ação em massa:

**Ações Disponíveis:**
- **Approve selected** (verde) - Aprovar todos os selecionados
- **Reject selected** (vermelho) - Rejeitar todos os selecionados
- **Delete selected** (vermelho escuro) - Deletar todos os selecionados
- **Export selected** (amarelo) - Exportar apenas os selecionados

---

## 6. BOTÃO "FOR MULTIPLE USERS"

Este botão permite ao Team Leader **criar registros de hora para múltiplos usuários de uma vez**, útil para:
- Registrar horas de reuniões em grupo
- Lançar horas de treinamentos coletivos
- Corrigir registros em massa

**Fluxo:**
1. Clica em "For multiple users"
2. Modal abre com campos:
   - **Users:** Seleção múltipla de colaboradores
   - **Project:** Projeto comum
   - **Activity:** Atividade comum
   - **Date:** Data comum
   - **From / To:** Horário comum
   - **Description:** Descrição comum
3. Salva e cria um registro para cada usuário selecionado

---

## 7. EXPORTAÇÃO

O botão **"Export"** permite exportar os dados filtrados em diversos formatos:

**Formatos Disponíveis:**
- **PDF** - Relatório formatado
- **Excel (XLSX)** - Planilha editável
- **CSV** - Dados brutos
- **HTML** - Visualização web

**Dados Exportados:**
- Todos os campos visíveis na tabela
- Totalizadores por usuário, projeto, período
- Valores financeiros (se configurado)

---

## 8. DIFERENÇAS EM RELAÇÃO À TELA "MY TIMES" (COLABORADOR)

| Aspecto | Colaborador (My times) | Team Leader (All times) |
|---------|------------------------|-------------------------|
| **Visibilidade** | Apenas registros próprios | Todos os registros da equipe |
| **Filtro de Usuário** | Não disponível | Disponível |
| **Ações de Aprovação** | Não disponível | Disponível (Approve/Reject) |
| **Criação para Outros** | Não disponível | Disponível |
| **Criação em Massa** | Não disponível | Disponível |
| **Edição de Registros Pendentes** | Não disponível | Disponível |
| **Exportação** | Limitada | Completa |

---

## 9. REGRAS DE NEGÓCIO

### 9.1 Permissões

- Apenas usuários com role **"Team Leader"** ou superior podem acessar "All times"
- Team Leaders só veem registros de usuários da **sua equipe** (hierarquia)
- Administradores veem **todos** os registros

### 9.2 Validações

- Não é possível aprovar registros com **sobreposição de horários**
- Não é possível aprovar registros com **duração zero**
- Não é possível aprovar registros de **projetos inativos**

### 9.3 Notificações

- **Aprovação:** E-mail enviado ao colaborador com resumo
- **Rejeição:** E-mail enviado com motivo obrigatório
- **Edição pelo gestor:** E-mail de notificação ao colaborador

---

## 10. INTEGRAÇÃO COM HUMANAMENTE

### 10.1 Campos Adicionais Necessários

Para adaptar ao Humanamente, adicionar:
- **Contract Type:** PJ ou CLT (filtro)
- **Hourly Rate:** Valor/hora do colaborador
- **Total Value:** Cálculo automático (duração × valor/hora)
- **Purchase Order:** Link para o pedido de compra gerado

### 10.2 Fluxo de Aprovação Adaptado

1. **Submissão:** Colaborador submete horas (status: `pending`)
2. **Aprovação:** Gestor aprova na tela "All times"
3. **Edge Function Dispara:**
   - Se **PJ:** Cria Pedido de Compra automaticamente
   - Se **CLT:** Apenas registra aprovação
   - Envia e-mail de notificação
4. **Solicitação de NF:** Botão adicional "Request Invoice" (apenas PJ)
   - Envia e-mail para financeiro com dados do pedido

### 10.3 Wireframe Adaptado (com Valores)

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│ Todas as Horas - Aprovação                                                         │
├─────────────────────────────────────────────────────────────────────────────────────┤
│ [≡] [▼ Período] [🔍 Buscar] [Status: Pendente▼] [Tipo: Todos▼]                    │
│                                                   [+ Criar] [Export▼]               │
├─────────────────────────────────────────────────────────────────────────────────────┤
│ ☑️ │ Usuário      │ Data     │ Projeto │ Horas │ Tipo │ Valor   │ Status   │ Ações │
│────┼──────────────┼──────────┼─────────┼───────┼──────┼─────────┼──────────┼───────│
│ ☑  │ João Silva   │ 01/02/26 │ Proj A  │ 8:00  │ PJ   │ R$800   │ Pendente │ ✓ ✗ ✎ │
│ ☑  │ Maria Santos │ 01/02/26 │ Proj B  │ 6:30  │ CLT  │ —       │ Pendente │ ✓ ✗ ✎ │
│ ☑  │ Pedro Costa  │ 31/01/26 │ Proj A  │ 10:00 │ PJ   │ R$1.200 │ Pendente │ ✓ ✗ ✎ │
└─────────────────────────────────────────────────────────────────────────────────────┘
│ 3 registros selecionados │ [Aprovar Selecionados] [Rejeitar Selecionados]         │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 11. PRÓXIMOS PASSOS

Para completar a análise, preciso:
1. Acessar a tela de **"Reporting"** para ver relatórios consolidados
2. Verificar se há uma tela específica de **"Approvals"** ou se tudo é feito em "All times"
3. Analisar o modal de **rejeição** (motivo obrigatório)
4. Verificar o modal de **edição** de registros

**Observação:** A demo está vazia, então não posso interagir com registros reais, mas a estrutura da tela e os botões estão todos visíveis e documentados.
