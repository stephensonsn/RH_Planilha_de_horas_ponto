# Análise Detalhada: Modal "Criar" Registro de Tempo - Kimai

**Acionamento:** Botão "+ Criar" na toolbar da tela "Meus horários"  
**Tipo:** Modal centralizado com overlay escuro

---

## 1. ESTRUTURA DO MODAL

### 1.1 Header do Modal

**Elementos:**

1. **Título:** "Criar"
   - Posição: Canto superior esquerdo
   - Fonte: Bold, ~18px
   - Cor: Branco

2. **Botão de Ajuda:** "?" (Index 3)
   - Posição: Ao lado do título
   - Cor: Cinza claro
   - Função: Abre tooltip ou documentação contextual

3. **Botão Fechar:** "×" (Index 4)
   - Posição: Canto superior direito
   - Cor: Cinza
   - Tooltip: "Fechar"
   - Função: Fecha o modal sem salvar

### 1.2 Corpo do Modal

**Layout:** Formulário vertical com campos empilhados  
**Largura:** ~600px  
**Fundo:** Escuro (#2c3e50)  
**Padding:** 24px

---

## 2. CAMPOS DO FORMULÁRIO

### Campo 1: "De" (Data e Hora de Início)

**Label:** "De" (Index 5)  
**Obrigatório:** Sim (indicado por asterisco vermelho implícito)

**Sub-campos:**

1. **Data** (Index 7)
   - Tipo: Date picker
   - Placeholder: "DD/MM/YYYY"
   - Valor padrão: Data atual (01/02/2026)
   - Ícone: Calendário (Index 6)
   - Largura: ~50% do campo

2. **Hora** (Index 9)
   - Tipo: Time picker
   - Placeholder: "HH:mm"
   - Valor padrão: Hora atual (11:14)
   - Ícone: Relógio (Index 8)
   - Largura: ~50% do campo

**Botão de Ação Rápida:** (Index 10)
- Ícone: Play/Now
- Função: Define data/hora para "agora"
- Cor: Verde

**Layout:** Inline (data e hora lado a lado)

---

### Campo 2: "Duração / Fim"

**Label:** "Duração / Fim" (Index 11)  
**Tipo:** Dual input (duração OU hora de fim)

**Sub-campos:**

1. **Duração** (Index 12)
   - Tipo: Time input
   - Placeholder: "0:00"
   - Valor padrão: 0:00
   - Formato aceito: HH:mm ou decimal (ex: 2.5)
   - Ícone: Cronômetro (Index 13)
   - Largura: ~50% do campo

2. **Fim** (Index 15)
   - Tipo: Time picker
   - Placeholder: "HH:mm"
   - Ícone: Relógio (Index 14)
   - Largura: ~50% do campo

**Botão de Ação Rápida:** (Index 16)
- Ícone: Stop/Now
- Função: Define hora de fim para "agora"
- Cor: Vermelho

**Comportamento:**
- Se "Duração" for preenchida, "Fim" é calculado automaticamente
- Se "Fim" for preenchido, "Duração" é calculada automaticamente
- Ambos podem ser deixados vazios para criar um timer ativo (em andamento)

**Layout:** Inline (duração e fim lado a lado)

---

### Campo 3: "Projeto"

**Label:** "Projeto" (Index 17)  
**Obrigatório:** Sim (asterisco vermelho)

**Tipo:** Combobox/Autocomplete (Index 19)  
**Características:**
- Busca em tempo real (typeahead)
- Dropdown com lista de projetos disponíveis
- Permite criar novo projeto (se permissão)
- Ícone: Pasta/Projeto (Index 18)

**Informações Exibidas:**
- Nome do projeto
- Cliente associado (se houver)
- Cor do projeto (badge colorido)

**Largura:** 100% do modal

---

### Campo 4: "Atividade"

**Label:** "Atividade" (Index 20)  
**Obrigatório:** Sim (asterisco vermelho)

**Tipo:** Combobox/Autocomplete (Index 22)  
**Características:**
- Busca em tempo real (typeahead)
- Dropdown com lista de atividades disponíveis
- Filtrado pelo projeto selecionado (dependência)
- Permite criar nova atividade (se permissão)
- Ícone: Tag/Etiqueta (Index 21)

**Informações Exibidas:**
- Nome da atividade
- Descrição curta (se houver)

**Largura:** 100% do modal

**Dependência:** Desabilitado até que um projeto seja selecionado

---

### Campo 5: "Descrição"

**Label:** "Descrição" (Index 23)  
**Obrigatório:** Não

**Tipo:** Textarea (Index 24)  
**Características:**
- Texto livre, multi-linha
- Placeholder: (vazio)
- Altura: ~80px (3-4 linhas)
- Redimensionável verticalmente

**Largura:** 100% do modal

---

### Campo 6: "Tags"

**Label:** "Tags" (Index 25)  
**Obrigatório:** Não

**Tipo:** Multi-select Combobox (Index 27)  
**Características:**
- Busca em tempo real (typeahead)
- Permite múltiplas seleções
- Permite criar novas tags
- Tags selecionadas aparecem como badges coloridos
- Ícone: Etiqueta (Index 26)

**Largura:** 100% do modal

---

## 3. FOOTER DO MODAL

**Posição:** Parte inferior do modal  
**Layout:** Flex, botões alinhados à direita

**Botões:**

1. **Salvar** (Index 28)
   - Cor: Azul (#3498db)
   - Texto: "Salvar"
   - Função: Valida e salva o registro
   - Estado: Desabilitado se campos obrigatórios não preenchidos

2. **Fechar** (Index 29)
   - Cor: Cinza (#95a5a6)
   - Texto: "Fechar"
   - Função: Fecha o modal sem salvar
   - Confirmação: Pergunta se há mudanças não salvas

---

## 4. VALIDAÇÕES

### Validações de Campo

1. **Projeto:** Obrigatório. Deve ser selecionado da lista.
2. **Atividade:** Obrigatório. Deve ser selecionado da lista (filtrado por projeto).
3. **Data de Início:** Obrigatória. Não pode ser futura (depende de configuração).
4. **Hora de Início:** Obrigatória.
5. **Duração/Fim:** Opcional para criar timer ativo. Se preenchido, fim não pode ser antes do início.
6. **Duração Máxima:** Não pode exceder limite configurado (ex: 12 horas).

### Validações de Negócio

1. **Sobreposição:** Não pode sobrepor outro registro do mesmo usuário.
2. **Projeto Ativo:** Projeto deve estar ativo para permitir apontamento.
3. **Permissões:** Usuário deve ter permissão para apontar no projeto selecionado.

### Feedback Visual

- **Erro:** Campo com borda vermelha + mensagem abaixo
- **Sucesso:** Campo com borda verde
- **Carregando:** Spinner no botão "Salvar"

---

## 5. WIREFRAME ASCII DO MODAL

```
┌───────────────────────────────────────────────────────────┐
│ Criar                                              ? × [4] │
├───────────────────────────────────────────────────────────┤
│                                                            │
│  De *                                                      │
│  ┌──────────────────┬──────────────────┬─────┐            │
│  │ 📅 01/02/2026    │ 🕐 11:14         │ ▶ [10]           │
│  └──────────────────┴──────────────────┴─────┘            │
│                                                            │
│  Duração / Fim                                             │
│  ┌──────────────────┬──────────────────┬─────┐            │
│  │ ⏱ 0:00          │ 🕐 HH:mm         │ ⏹ [16]           │
│  └──────────────────┴──────────────────┴─────┘            │
│                                                            │
│  Projeto *                                                 │
│  ┌────────────────────────────────────────────────────┐   │
│  │ 🔽 Selecione um projeto...                         │   │
│  └────────────────────────────────────────────────────┘   │
│                                                            │
│  Atividade *                                               │
│  ┌────────────────────────────────────────────────────┐   │
│  │ 🔽 Selecione uma atividade...                      │   │
│  └────────────────────────────────────────────────────┘   │
│                                                            │
│  Descrição                                                 │
│  ┌────────────────────────────────────────────────────┐   │
│  │                                                     │   │
│  │                                                     │   │
│  └────────────────────────────────────────────────────┘   │
│                                                            │
│  Tags                                                      │
│  ┌────────────────────────────────────────────────────┐   │
│  │ 🔽 Adicione tags...                                │   │
│  └────────────────────────────────────────────────────┘   │
│                                                            │
│                                    ┌────────┬──────────┐   │
│                                    │ Salvar │  Fechar  │   │
│                                    └────────┴──────────┘   │
└───────────────────────────────────────────────────────────┘
```

---

## 6. FLUXO DE INTERAÇÃO

### Fluxo 1: Criar Registro Completo (Passado)

1. Usuário clica em "+ Criar"
2. Modal abre com data/hora atual pré-preenchidas
3. Usuário seleciona **Projeto** (dropdown)
4. Campo **Atividade** é habilitado
5. Usuário seleciona **Atividade** (dropdown)
6. Usuário preenche **Duração** (ex: 2:30) OU **Fim** (ex: 13:44)
7. Sistema calcula automaticamente o campo complementar
8. Usuário preenche **Descrição** (opcional)
9. Usuário adiciona **Tags** (opcional)
10. Usuário clica em **Salvar**
11. Sistema valida, salva e fecha o modal
12. Tabela é atualizada com o novo registro

### Fluxo 2: Iniciar Timer Ativo (Presente)

1. Usuário clica em "+ Criar"
2. Modal abre com data/hora atual pré-preenchidas
3. Usuário seleciona **Projeto**
4. Usuário seleciona **Atividade**
5. Usuário **NÃO** preenche Duração/Fim (deixa vazio)
6. Usuário clica em **Salvar**
7. Sistema cria registro com status "Em andamento"
8. Timer no header global começa a contar
9. Modal fecha
10. Tabela mostra registro com status "Ativo" (sem hora de fim)

### Fluxo 3: Uso do Botão "Agora"

1. Usuário está preenchendo o formulário
2. Usuário clica no botão ▶ (Now) ao lado do campo "De"
3. Sistema preenche data/hora com o momento atual
4. OU usuário clica no botão ⏹ (Stop) ao lado do campo "Fim"
5. Sistema preenche hora de fim com o momento atual
6. Duração é calculada automaticamente

---

## 7. OBSERVAÇÕES PARA HUMANAMENTE

### Adaptações Necessárias

1. **Campos Adicionais:**
   - Campo "Cliente" (se não estiver implícito no Projeto)
   - Campo "Valor/Hora" (se não for fixo por projeto)
   - Campo "Faturável" (checkbox)

2. **Integração com Aprovação:**
   - Campo "Status" (Draft, Pendente, Aprovado)
   - Botão "Submeter para Aprovação" (além de Salvar)

3. **Visibilidade de Valores:**
   - Toggle para mostrar/ocultar valor projetado
   - Cálculo em tempo real: Duração × Valor/Hora

4. **Geolocalização (CLT):**
   - Não aplicável para PJ, mas considerar para CLT

### Melhorias Sugeridas

1. **Atalhos de Teclado:**
   - Ctrl+S para salvar
   - Esc para fechar
   - Tab para navegar entre campos

2. **Histórico Rápido:**
   - Botão "Repetir último" para preencher com dados do último registro

3. **Templates:**
   - Salvar combinações frequentes de Projeto+Atividade+Descrição

4. **Validação em Tempo Real:**
   - Feedback visual imediato (borda verde/vermelha)
   - Mensagens de erro claras e contextuais
