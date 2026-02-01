# Análise Detalhada: Tela "Calendário" - Kimai

**URL:** https://demo-empty.kimai.org/pt_BR/calendar/?date=2026-01-26&view=month  
**Biblioteca:** FullCalendar (provavelmente)

---

## 1. ESTRUTURA DA TELA

### 1.1 Header da Página

**Título:** "Calendário"  
**Posição:** Topo da área de conteúdo

### 1.2 Barra de Controles do Calendário

**Posição:** Logo abaixo do título  
**Layout:** Flex, itens distribuídos horizontalmente

**Elementos da Esquerda para Direita:**

1. **Título do Período Atual** (centro-esquerda)
   - Texto: "fevereiro de 2026"
   - Fonte: Bold, ~24px
   - Cor: Branco

2. **Botões de Visualização** (centro)
   - **Mês** (Index 14) - Ativo (azul)
   - **Semana** (Index 15) - Inativo (cinza)
   - **Dia** (Index 16) - Inativo (cinza)
   - Estilo: Botões agrupados (button group)
   - Cor ativo: #3498db (azul)

3. **Botão "Hoje"** (centro-direita)
   - Texto: "Hoje"
   - Cor: Azul (#3498db)
   - Função: Navega para o dia atual

4. **Botões de Navegação** (direita)
   - **Anterior** (Index 17) - Seta <
   - **Próximo** (Index 18) - Seta >
   - Cor: Azul (#3498db)
   - Função: Navega entre meses/semanas/dias

---

## 2. VISUALIZAÇÃO DO CALENDÁRIO (MÊS)

### 2.1 Header do Calendário

**Dias da Semana:**
- seg. (segunda-feira) - Index 19
- ter. (terça-feira) - Index 20
- qua. (quarta-feira) - Index 21
- qui. (quinta-feira) - Index 22
- sex. (sexta-feira) - Index 23
- sáb. (sábado) - Index 24
- dom. (domingo) - Index 25

**Estilo:**
- Fundo: Escuro (#34495e)
- Texto: Branco, uppercase
- Abreviação de 3 letras + ponto

### 2.2 Coluna de Semanas

**Posição:** Primeira coluna à esquerda  
**Conteúdo:** Número da semana do ano

**Exemplos:**
- Sm5 (Semana 5) - Index 27
- Sm6 (Semana 6) - Index 35
- Sm7 (Semana 7) - Index 43
- ...

**Estilo:**
- Fundo: Levemente mais escuro
- Texto: Cinza claro
- Clicável (abre visualização semanal)

### 2.3 Células de Dias

**Estrutura de Cada Célula:**

1. **Número do Dia** (canto superior esquerdo)
   - Tamanho: ~16px
   - Cor: Branco (dias do mês atual), Cinza (dias de outros meses)
   - Clicável (abre visualização diária)

2. **Área de Eventos** (corpo da célula)
   - Espaço para exibir eventos/registros de tempo
   - Vazio nesta demo

**Estilo:**
- Fundo: #2c3e50 (dias do mês atual)
- Fundo: #1a252f (dias de outros meses - mais escuro)
- Borda: Sutil, cinza escuro
- Altura: ~100px (variável)
- Hover: Fundo levemente mais claro

**Interações:**
- **Clique simples:** Abre visualização diária
- **Clique duplo:** Abre modal para criar novo registro naquele dia
- **Arrastar:** Seleciona intervalo de datas (para criar registro)

---

## 3. VISUALIZAÇÃO SEMANAL (Não testada, mas inferida)

**Acionamento:** Botão "Semana" (Index 15)

**Estrutura Esperada:**
- 7 colunas (uma para cada dia da semana)
- Linhas de hora (00:00 a 23:59)
- Blocos coloridos representando registros de tempo
- Possibilidade de arrastar para criar/editar registros

---

## 4. VISUALIZAÇÃO DIÁRIA (Não testada, mas inferida)

**Acionamento:** Botão "Dia" (Index 16) ou clique em um dia

**Estrutura Esperada:**
- 1 coluna com o dia selecionado
- Linhas de hora (00:00 a 23:59) em intervalos de 30 min
- Blocos coloridos representando registros de tempo
- Visão detalhada de cada registro (projeto, atividade, duração)

---

## 5. EVENTOS/REGISTROS NO CALENDÁRIO

### 5.1 Representação Visual (quando há dados)

**Bloco de Evento:**
- Retângulo colorido
- Cor: Definida pelo projeto
- Altura: Proporcional à duração
- Largura: 100% da célula (ou parcial se houver sobreposição)

**Informações Exibidas:**
- Hora de início
- Nome do projeto
- Nome da atividade (se houver espaço)
- Duração (se houver espaço)

**Exemplo Visual:**
```
┌─────────────────────┐
│ 09:00 - IMEP        │
│ Desenvolvimento     │
│ 3h 30m              │
└─────────────────────┘
```

### 5.2 Interações com Eventos

1. **Clique:** Abre modal de edição do registro
2. **Arrastar:** Move o registro para outro dia/horário
3. **Redimensionar:** Altera a duração (arrastar borda inferior)
4. **Hover:** Exibe tooltip com detalhes completos

---

## 6. CRIAÇÃO DE REGISTROS NO CALENDÁRIO

### Método 1: Clique Duplo em um Dia

1. Usuário dá duplo clique em uma célula de dia
2. Modal "Criar" abre com a data pré-preenchida
3. Usuário preenche projeto, atividade, horário, etc.
4. Ao salvar, o evento aparece no calendário

### Método 2: Arrastar para Selecionar Intervalo

1. Usuário clica e arrasta sobre múltiplas células
2. Modal "Criar" abre com intervalo de datas selecionado
3. Usuário preenche os demais campos
4. Ao salvar, múltiplos eventos são criados (um por dia)

### Método 3: Arrastar na Visualização Semanal/Diária

1. Usuário clica e arrasta sobre as linhas de hora
2. Modal "Criar" abre com data e horário de início/fim pré-preenchidos
3. Usuário preenche projeto e atividade
4. Ao salvar, o evento aparece no calendário

---

## 7. WIREFRAME ASCII DO CALENDÁRIO (MÊS)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Calendário                                                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  fevereiro de 2026    [Mês] [Semana] [Dia]    [Hoje]    [<] [>]            │
│                                                                              │
├────┬────────┬────────┬────────┬────────┬────────┬────────┬────────┤        │
│    │ seg.   │ ter.   │ qua.   │ qui.   │ sex.   │ sáb.   │ dom.   │        │
├────┼────────┼────────┼────────┼────────┼────────┼────────┼────────┤        │
│ Sm5│   26   │   27   │   28   │   29   │   30   │   31   │    1   │        │
│    │        │        │        │        │        │        │        │        │
│    │        │        │        │        │        │        │        │        │
├────┼────────┼────────┼────────┼────────┼────────┼────────┼────────┤        │
│ Sm6│    2   │    3   │    4   │    5   │    6   │    7   │    8   │        │
│    │        │        │        │        │        │        │        │        │
│    │        │        │        │        │        │        │        │        │
├────┼────────┼────────┼────────┼────────┼────────┼────────┼────────┤        │
│ Sm7│    9   │   10   │   11   │   12   │   13   │   14   │   15   │        │
│    │        │        │        │        │        │        │        │        │
│    │        │        │        │        │        │        │        │        │
├────┼────────┼────────┼────────┼────────┼────────┼────────┼────────┤        │
│ Sm8│   16   │   17   │   18   │   19   │   20   │   21   │   22   │        │
│    │        │        │        │        │        │        │        │        │
│    │        │        │        │        │        │        │        │        │
├────┼────────┼────────┼────────┼────────┼────────┼────────┼────────┤        │
│ Sm9│   23   │   24   │   25   │   26   │   27   │   28   │    1   │        │
│    │        │        │        │        │        │        │        │        │
│    │        │        │        │        │        │        │        │        │
└────┴────────┴────────┴────────┴────────┴────────┴────────┴────────┘        │
```

---

## 8. OBSERVAÇÕES PARA HUMANAMENTE

### Adaptações Necessárias

1. **Cores dos Projetos:**
   - Cada projeto deve ter uma cor associada
   - Eventos no calendário usam a cor do projeto

2. **Status Visual:**
   - Registros "Draft": Borda tracejada
   - Registros "Pendente": Cor mais clara + ícone de relógio
   - Registros "Aprovado": Cor sólida + ícone de check
   - Registros "Rejeitado": Cor vermelha + ícone de X

3. **Indicadores de Valor:**
   - Se o toggle de visibilidade estiver ativo, exibir valor no tooltip
   - Exemplo: "R$ 350,00" abaixo da duração

4. **Filtros:**
   - Adicionar filtros por projeto, cliente, status
   - Barra de filtros acima do calendário

### Melhorias Sugeridas

1. **Legenda de Cores:**
   - Widget lateral com lista de projetos e suas cores
   - Checkbox para mostrar/ocultar projetos específicos

2. **Resumo Diário:**
   - Exibir total de horas do dia no canto da célula
   - Exemplo: "8h 30m" em fonte pequena

3. **Indicador de Metas:**
   - Se houver meta de horas por dia/semana, exibir progresso
   - Barra de progresso ou cor de fundo da célula

4. **Exportação:**
   - Botão para exportar calendário (PDF, iCal)
   - Útil para compartilhar com clientes

5. **Sincronização:**
   - Integração com Google Calendar / Outlook
   - Eventos externos aparecem em cor diferente
