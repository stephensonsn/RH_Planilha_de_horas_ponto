# Análise Detalhada: Tela "Meus horários" (Timesheet) - Kimai

**URL:** https://demo-empty.kimai.org/pt_BR/timesheet/  
**Estado:** Vazio (sem registros)

---

## 1. ESTRUTURA DA TELA

### 1.1 Header da Página

**Título:** "Meus horários"  
**Posição:** Topo da área de conteúdo

### 1.2 Barra de Ferramentas (Toolbar)

**Posição:** Logo abaixo do título  
**Fundo:** Faixa escura (#34495e)  
**Layout:** Flex, itens distribuídos horizontalmente

**Elementos da Esquerda para Direita:**

1. **Botão de Visualização em Grade** (Index 14)
   - Ícone: Grid/tabela
   - Tooltip: "Alterar a visibilidade da coluna"
   - Cor: Verde (#27ae60)
   - Função: Abre modal para mostrar/ocultar colunas da tabela

2. **Dropdown de Filtro de Período** (Index 16)
   - Ícone: Funil (filter)
   - Cor: Amarelo (#f39c12)
   - Tooltip: "Filtro de busca"
   - Opções visíveis:
     - **Intervalo de tempo**
     - Período total
     - Hoje
     - Ontem
     - Essa semana
     - Última semana
     - Este ano — até hoje
     - fevereiro 2026
     - janeiro 2026
     - dezembro 2025
     - novembro 2025
     - T1 2026 (trimestre)
     - ...

3. **Botão de Busca Avançada** (Index 15)
   - Ícone: Lupa com engrenagem
   - Cor: Vermelho (#e74c3c)
   - Função: Abre painel de filtros avançados

4. **Campo de Busca Rápida** (Index 17-18)
   - Input de texto
   - Placeholder: "Procurar"
   - Ícone de lupa à direita
   - Cor: Amarelo (#f39c12)
   - Busca em descrição, tags, etc.

**Elementos da Direita:**

5. **Botão "Criar"** (Index 19)
   - Ícone: "+" (plus)
   - Texto: "Criar"
   - Cor: Verde (#27ae60)
   - Função: Abre modal para criar novo registro de tempo

6. **Botão "Exportar"** (Index 20)
   - Ícone: Download
   - Texto: "Exportar"
   - Cor: Amarelo (#f39c12)
   - Dropdown com opções de formato (PDF, Excel, CSV, etc.)

7. **Botão de Ajuda** (Index 21)
   - Ícone: "?"
   - Cor: Cinza
   - Função: Abre documentação/ajuda contextual

---

## 2. ÁREA DE CONTEÚDO (TABELA)

### 2.1 Estado Vazio

**Mensagem exibida:**
> ℹ️ Não foram encontradas entradas com base nos filtros selecionados.

**Estilo:**
- Ícone de informação (azul)
- Texto centralizado
- Fundo levemente mais claro

### 2.2 Estrutura da Tabela (quando há dados)

**Colunas Padrão:**

1. **Checkbox** (seleção múltipla)
2. **Data** (formato: dd/mm/yyyy)
3. **Início** (hora de início)
4. **Fim** (hora de fim)
5. **Duração** (hh:mm)
6. **Cliente** (nome do cliente)
7. **Projeto** (nome do projeto)
8. **Atividade** (tipo de atividade)
9. **Descrição** (texto livre)
10. **Tags** (etiquetas coloridas)
11. **Valor** (R$ ou moeda configurada)
12. **Ações** (botões de editar, deletar, etc.)

**Observação:** As colunas são configuráveis via botão de visualização em grade.

---

## 3. INTERAÇÕES E FUNCIONALIDADES

### 3.1 Filtros

**Filtro de Período (Dropdown Amarelo):**
- Permite seleção rápida de períodos pré-definidos
- Opções: Hoje, Ontem, Essa semana, Última semana, Este mês, etc.
- Também permite seleção de meses específicos
- Trimestres (T1, T2, T3, T4)

**Busca Avançada (Botão Vermelho):**
- Abre painel lateral ou modal com filtros detalhados:
  - Cliente
  - Projeto
  - Atividade
  - Tags
  - Status (ativo, parado, exportado)
  - Faturável (sim/não)
  - Usuário (para gestores)

**Busca Rápida (Campo de Texto):**
- Busca em tempo real
- Campos pesquisados: descrição, tags, notas

### 3.2 Ações em Massa

**Seleção Múltipla:**
- Checkbox na primeira coluna de cada linha
- Checkbox no header para selecionar todos
- Quando itens são selecionados, aparece barra de ações:
  - Editar em massa
  - Deletar selecionados
  - Exportar selecionados
  - Marcar como faturável/não faturável

### 3.3 Ações Individuais

**Botões na coluna "Ações":**
1. **Reiniciar** (ícone de play circular)
   - Inicia novo timer com os mesmos dados
2. **Editar** (ícone de lápis)
   - Abre modal de edição
3. **Duplicar** (ícone de cópia)
   - Cria cópia do registro
4. **Deletar** (ícone de lixeira)
   - Confirmação antes de deletar

---

## 4. BOTÃO "CRIAR" - FLUXO PRINCIPAL

### 4.1 Ação ao Clicar

Abre **modal** para criar novo registro de tempo.

### 4.2 Campos do Modal (a ser analisado)

- Projeto (dropdown obrigatório)
- Atividade (dropdown obrigatório)
- Data (date picker)
- Início (time picker)
- Fim (time picker)
- Duração (calculada automaticamente ou manual)
- Descrição (textarea)
- Tags (multi-select)
- Faturável (checkbox)

---

## 5. WIREFRAME ASCII DA TELA

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER GLOBAL (Timer: 0:00 | Reiniciar | Avatar: John Doe)                 │
├─────────────────────────────────────────────────────────────────────────────┤
│ SIDEBAR │ Meus horários                                                     │
│         ├───────────────────────────────────────────────────────────────────┤
│ ☰ Menu  │ [🔲] [🔽 Período] [🔍+] [Procurar...] 🔍    [+ Criar] [⬇ Exportar]│
│         ├───────────────────────────────────────────────────────────────────┤
│ • Dash  │                                                                   │
│ • Tempo │  ℹ️ Não foram encontradas entradas com base nos filtros           │
│   - Meus│     selecionados.                                                 │
│   - Sem │                                                                   │
│   - Cal │                                                                   │
│ • Contr │                                                                   │
│ • Relat │                                                                   │
│ • Desp  │                                                                   │
│ • Taref │                                                                   │
│         │                                                                   │
│         │                                                                   │
│         │                                                                   │
└─────────┴───────────────────────────────────────────────────────────────────┘
```

---

## 6. PRÓXIMOS PASSOS

1. Clicar em "Criar" para analisar o modal de criação
2. Criar um registro de exemplo
3. Analisar a tabela com dados
4. Testar fluxo de edição
5. Verificar se há fluxo de aprovação disponível
