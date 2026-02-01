# Análise Detalhada e Wireframes: Interface PJ (Kimai)

**Projeto:** Humanamente - Adaptação da Interface para Terceiros (PJ)  
**Baseado em:** Demo Interativa Kimai (john_user)  
**Autor:** Manus AI  
**Data:** 01 de Fevereiro de 2026

---

## SUMÁRIO EXECUTIVO

Este documento detalha a arquitetura visual e funcional completa da interface de apontamento de horas para colaboradores PJ, baseada na análise da demo do Kimai. O objetivo é fornecer um guia exato para a implementação no Humanamente, garantindo fidelidade ao fluxo e à experiência do usuário de referência.

**Estrutura do Documento:**
1.  **Estrutura Visual Global:** Header, Sidebar e Paleta de Cores.
2.  **Wireframe: Dashboard:** Análise dos widgets e layout inicial.
3.  **Wireframe: Tela de Apontamentos (Timesheet):** Detalhamento da tabela e barra de ferramentas.
4.  **Wireframe: Modal de Criação/Edição:** Todos os campos e validações.
5.  **Wireframe: Tela de Calendário:** Visualização mensal e interações.
6.  **Fluxo de Usuário Completo:** Do login à submissão para aprovação.

---

## 1. ESTRUTURA VISUAL GLOBAL

### 1.1 Header Superior (Barra de Topo)

- **Layout:** Fixo, altura de ~60px, fundo escuro (`#2c3e50`).
- **Elementos:**
  - **Logo:** Canto esquerdo.
  - **Timer Ativo:** Botão de play/stop com contador.
  - **Botão Reiniciar:** Ícone de histórico para repetir atividades recentes.
  - **Menu Pessoal:** Avatar, nome do usuário e dropdown para `Configurações` e `Sair`.

### 1.2 Sidebar de Navegação (Esquerda)

- **Layout:** Fixo, largura de ~240px, fundo escuro (`#34495e`).
- **Menu Principal (PJ):**
  - **Dashboard:** Visão geral e resumos.
  - **Controle de tempo (expansível):**
    - **Meus horários:** Tela principal de apontamentos (tabela).
    - **Horas de trabalhos semanais:** (Página indisponível na demo, mas deve ser uma tabela resumida por dia).
    - **Calendário:** Visão de calendário com eventos.
  - **Relatórios:** (Fora do escopo inicial).
  - **Despesas:** (Fora do escopo inicial).
  - **Tarefas:** (Fora do escopo inicial).

### 1.3 Paleta de Cores (Tema Escuro Kimai)

- **Fundo Principal:** `#2c3e50` (Azul-ardósia escuro)
- **Sidebar/Header:** `#34495e` (Azul-ardósia mais claro)
- **Primária (Ações Positivas):** `#27ae60` (Verde)
- **Secundária (Navegação/Links):** `#3498db` (Azul)
- **Alerta/Timer Ativo:** `#e74c3c` (Vermelho)
- **Filtros/Exportação:** `#f39c12` (Amarelo)
- **Texto Principal:** `#ecf0f1` (Branco/Cinza muito claro)

---

## 2. WIREFRAME: DASHBOARD

**Objetivo:** Fornecer uma visão rápida e resumida da carga de trabalho e progresso.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER GLOBAL (Timer: 0:00 | Reiniciar | Avatar: John Doe)                 │
├─────────────────────────────────────────────────────────────────────────────┤
│ SIDEBAR │ Dashboard                                                         │
│         ├───────────────────────────────────────────────────────────────────┤
│ ☰ Menu  │ ┌───────────────────────────────────────────────────────────────┐ │
│         │ │ Seu horário de trabalho                     [<] [Semana 05] [>] │ │
│ • Dash  │ │ ┌───────────────────────────────────────────────────────────┐ │ │
│ • Tempo │ │ │ [Gráfico de Linha: Horas por Dia na Semana]               │ │ │
│         │ │ └───────────────────────────────────────────────────────────┘ │ │
│         │ └───────────────────────────────────────────────────────────────┘ │
│         │ ┌───────────┬───────────┬───────────┬───────────┐               │
│         │ │ 0:00      │ 0:00      │ 0:00      │ 0:00      │               │
│         │ │ Hoje      │ Semana 05 │ Janeiro   │ Ano 2026  │               │
│         │ └───────────┴───────────┴───────────┴───────────┘               │
│         │ ┌──────────────────────────┬──────────────────────────┐          │
│         │ │ Minhas tarefas      [+]  │ Tarefas pendentes   [?]  │          │
│         │ │                          │                          │          │
│         │ │           1              │           1              │          │
│         │ └──────────────────────────┴──────────────────────────┘          │
└─────────┴───────────────────────────────────────────────────────────────────┘
```

### Detalhes dos Widgets:

1.  **Gráfico "Seu horário de trabalho":**
    - **Tipo:** Gráfico de linha.
    - **Eixo X:** Dias da semana/mês.
    - **Eixo Y:** Total de horas apontadas.
    - **Interação:** Navegação por setas para mudar o período (semana/mês anterior/próximo).

2.  **Cards de Resumo:**
    - **Conteúdo:** Exibem o total de horas para "Hoje", "Semana Atual", "Mês Atual" e "Ano Atual".
    - **Função:** Visão imediata dos totais acumulados.

3.  **Widgets de Tarefas:**
    - **Função:** Gerenciamento simples de tarefas (fora do escopo principal de horas, mas parte do layout).

---

## 3. WIREFRAME: TELA DE APONTAMENTOS ("Meus horários")

**Objetivo:** Tela central para listar, filtrar, criar e gerenciar todos os registros de tempo.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER GLOBAL (Timer: 0:00 | Reiniciar | Avatar: John Doe)                 │
├─────────────────────────────────────────────────────────────────────────────┤
│ SIDEBAR │ Meus horários                                                     │
│         ├───────────────────────────────────────────────────────────────────┤
│ ☰ Menu  │ [🔲 Colunas] [🔽 Período] [🔍 Filtros] [Procurar...]    [+ Criar] [⬇ Exportar]│
│         ├───────────────────────────────────────────────────────────────────┤
│         │ ┌─┬────┬──────┬─────┬───────┬─────────┬─────────┬─────────┬─────┐ │
│         │ │✅│Data│Início│Fim  │Duração│Projeto  │Atividade│Descrição│Ações│ │
│         │ ├─┼────┼──────┼─────┼───────┼─────────┼─────────┼─────────┼─────┤ │
│         │ │ │... │ ...  │ ... │ ...   │ ...     │ ...     │ ...     │▶ ✏️🗑️│ │
│         │ └─┴────┴──────┴─────┴───────┴─────────┴─────────┴─────────┴─────┘ │
│         │                                                                   │
│         │ ┌───────────────────────────────────────────────────────────────┐ │
│         │ │ Ações em Massa: [Editar] [Exportar] [Submeter p/ Aprovação]   │ │
│         │ └───────────────────────────────────────────────────────────────┘ │
└─────────┴───────────────────────────────────────────────────────────────────┘
```

### Detalhes da Barra de Ferramentas:

-   **Botão Colunas:** Abre modal para mostrar/ocultar colunas da tabela.
-   **Dropdown Período:** Filtro rápido por `Hoje`, `Esta Semana`, `Este Mês`, etc.
-   **Botão Filtros:** Abre painel de filtros avançados (por `Projeto`, `Atividade`, `Tags`, `Status`).
-   **Campo Procurar:** Busca textual na `Descrição` e `Tags`.
-   **Botão "+ Criar":** Abre o modal de criação de registro.
-   **Botão Exportar:** Exporta os dados filtrados para PDF, CSV, Excel.

### Detalhes da Tabela:

-   **Seleção:** Checkbox na primeira coluna para ações em massa.
-   **Ações em Massa:** Barra que surge ao selecionar itens, com botões para `Editar`, `Exportar` e, crucialmente, **`Submeter para Aprovação`**. Este botão muda o `status` dos registros de `draft` para `pending`.
-   **Ações por Linha:**
    -   `▶` (Reiniciar): Inicia um novo timer com os mesmos dados.
    -   `✏️` (Editar): Abre o modal de edição com os dados da linha.
    -   `🗑️` (Deletar): Remove o registro (apenas se `status` for `draft`).

---

## 4. WIREFRAME: MODAL DE CRIAÇÃO/EDIÇÃO

**Objetivo:** Interface única para inserir ou modificar um registro de tempo.

```
┌───────────────────────────────────────────────────────────┐
│ Criar Registro de Tempo                            ? ×    │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  De *                Duração / Fim                        │
│  [📅 Data] [🕐 Hora]   [⏱ Duração] [🕐 Fim]                  │
│                                                           │
│  Projeto *                                                │
│  [🔽 Selecione um projeto...]                             │
│                                                           │
│  Atividade *                                              │
│  [🔽 Selecione uma atividade...]                          │
│                                                           │
│  Descrição                                                │
│  [Caixa de texto para detalhes do trabalho...]            │
│                                                           │
│  Tags                      Faturável? [✅]                │
│  [🔽 Adicione tags...]                                    │
│                                                           │
│                                    ┌────────┬──────────┐  │
│                                    │ Salvar │  Fechar  │  │
│                                    └────────┴──────────┘  │
└───────────────────────────────────────────────────────────┘
```

### Detalhes dos Campos:

-   **De (Início):** Data e hora de início. Obrigatório.
-   **Duração / Fim:** Preencher um calcula o outro. Deixar ambos vazios cria um timer ativo.
-   **Projeto:** Dropdown com busca. Obrigatório.
-   **Atividade:** Dropdown com busca, filtrado pelo projeto. Obrigatório.
-   **Descrição:** Campo de texto para detalhar o trabalho realizado.
-   **Tags:** Campo de seleção múltipla para categorização.
-   **Faturável:** Checkbox (padrão: `true`) para indicar se a hora deve ser cobrada.

### Botões do Modal:

-   **Salvar:** Salva o registro com status `draft`. Não submete para aprovação.
-   **Fechar:** Descarta as alterações.

---

## 5. WIREFRAME: TELA DE CALENDÁRIO

**Objetivo:** Visualizar os apontamentos em um formato de calendário, permitindo uma visão mais intuitiva da distribuição do trabalho.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Calendário                                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│  fevereiro de 2026    [Mês] [Semana] [Dia]    [Hoje]    [<] [>]           │
├────┬────────┬────────┬────────┬────────┬────────┬────────┬────────┤       │
│    │ seg.   │ ter.   │ qua.   │ qui.   │ sex.   │ sáb.   │ dom.   │       │
├────┼────────┼────────┼────────┼────────┼────────┼────────┼────────┤       │
│ Sm5│   26   │   27   │   28   │   29   │   30   │   31   │    1   │       │
│    │        │        │ [🟦 2h] │        │        │        │        │       │
├────┼────────┼────────┼────────┼────────┼────────┼────────┼────────┤       │
│ Sm6│    2   │    3   │    4   │    5   │    6   │    7   │    8   │       │
│    │ [🟩 8h] │ [🟩 4h] │        │        │        │        │        │       │
└────┴────────┴────────┴────────┴────────┴────────┴────────┴────────┘       │
```

### Detalhes e Interações:

-   **Visualização:** Botões para alternar entre `Mês`, `Semana` e `Dia`.
-   **Eventos:** Registros de tempo aparecem como blocos coloridos. A cor é definida pelo projeto.
-   **Interação:**
    -   **Clicar em um dia:** Abre o modal de criação com a data pré-selecionada.
    -   **Clicar em um evento:** Abre o modal de edição para aquele registro.
    -   **Arrastar e soltar:** Permite mover um registro para outro dia.
    -   **Redimensionar (na visão semanal/diária):** Permite ajustar a duração do registro.
-   **Status Visual:** A borda ou opacidade do evento pode indicar seu status (`draft`, `pending`, `approved`).

---

## 6. FLUXO DE USUÁRIO COMPLETO (APONTAMENTO E APROVAÇÃO)

**Persona:** Colaborador PJ

1.  **Login:** O usuário acessa o Humanamente e vê o **Dashboard** com seus resumos.

2.  **Iniciar Trabalho (Método 1: Timer Ativo):**
    a. Clica no botão de `Play` (▶) no header.
    b. Abre o **Modal de Criação**.
    c. Seleciona `Projeto` e `Atividade`.
    d. Deixa `Duração` e `Fim` vazios.
    e. Clica em `Salvar`.
    f. O modal fecha e o timer no header começa a contar.

3.  **Lançar Trabalho (Método 2: Manual):**
    a. Navega para **Meus horários**.
    b. Clica em `+ Criar`.
    c. Preenche todos os campos no **Modal de Criação**, incluindo `Duração` ou `Fim`.
    d. Clica em `Salvar`.
    e. O registro aparece na tabela com status `draft`.

4.  **Finalizar Trabalho (Timer Ativo):**
    a. Clica no botão de `Stop` (⏹️) no header.
    b. Abre o **Modal de Edição** do registro ativo.
    c. A `Duração` e `Fim` já estão preenchidos.
    d. O usuário pode adicionar `Descrição` e `Tags`.
    e. Clica em `Salvar`.

5.  **Revisão e Submissão (Final da Semana/Mês):**
    a. Na tela **Meus horários**, o usuário revisa todos os seus lançamentos (`draft`).
    b. Seleciona, via checkbox, todos os registros que deseja submeter.
    c. A barra de **Ações em Massa** aparece.
    d. Clica no botão **`Submeter para Aprovação`**.
    e. O sistema pede confirmação: "Você tem certeza que deseja submeter X registros? Após a submissão, eles não poderão ser editados."
    f. Ao confirmar, o `status` de todos os registros selecionados muda de `draft` para `pending`.

6.  **Notificação para o Gestor:**
    a. A mudança de status para `pending` dispara uma **Edge Function** no Supabase.
    b. A função envia um e-mail para o gestor responsável: "[Nome do Colaborador] submeteu horas para sua aprovação."

7.  **Fluxo de Aprovação (Visão do Gestor):**
    a. O gestor acessa uma tela de **"Aprovações"**.
    b. Vê uma lista de todos os registros com status `pending`, agrupados por colaborador.
    c. O gestor pode:
        i. **Aprovar em Massa:** Selecionar todos os registros de um colaborador e clicar em `Aprovar`.
        ii. **Rejeitar em Massa:** Selecionar registros e clicar em `Rejeitar`, adicionando um motivo.
        iii. **Aprovar/Rejeitar Individualmente**.
    d. Registros aprovados mudam o status para `approved` e disparam o fluxo financeiro (criação de pedido de compra).
    e. Registros rejeitados mudam o status para `rejected` e notificam o colaborador por e-mail para que ele possa corrigir e ressubmeter (criando um novo registro).
