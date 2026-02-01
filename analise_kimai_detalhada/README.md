# Análise Detalhada: Interface PJ (Kimai) para Humanamente

**Projeto:** Humanamente - Módulo de Apontamento de Horas para Terceiros (PJ)  
**Baseado em:** Demo Interativa Kimai (https://demo-empty.kimai.org)  
**Credenciais de Acesso:** `john_user` / `password`  
**Data da Análise:** 01 de Fevereiro de 2026

---

## 📋 Conteúdo do Diretório

Este diretório contém uma análise completa e detalhada da interface do Kimai, capturada através de interação direta com a demonstração ao vivo. Todos os wireframes, campos, botões, fluxos e regras de negócio foram documentados para garantir fidelidade na implementação no Humanamente.

### Documentos Incluídos:

1.  **`analise_completa_kimai_pj.md`** - **DOCUMENTO MESTRE**
    -   Sumário executivo com todos os wireframes principais.
    -   Estrutura visual global (header, sidebar, paleta de cores).
    -   Wireframes ASCII art de todas as telas principais.
    -   Fluxo de usuário completo (do login à aprovação).

2.  **`analise_kimai_dashboard.md`**
    -   Análise detalhada do Dashboard.
    -   Widgets, gráficos e cards de resumo.
    -   Interações e navegação.

3.  **`analise_kimai_timesheet.md`**
    -   Análise da tela "Meus horários" (tabela principal).
    -   Barra de ferramentas completa (botões, filtros, busca).
    -   Estrutura da tabela e colunas.
    -   Ações em massa e ações por linha.

4.  **`analise_kimai_modal_criar.md`**
    -   Análise completa do modal de criação/edição de registros.
    -   Todos os campos, tipos de input e validações.
    -   Fluxos de interação (timer ativo, lançamento manual).
    -   Wireframe ASCII art do modal.

5.  **`analise_kimai_calendario.md`**
    -   Análise da tela de Calendário.
    -   Visualizações (mês, semana, dia).
    -   Interações (criar, editar, arrastar eventos).
    -   Representação visual de eventos.

---

## 🎯 Objetivo da Análise

Fornecer um guia técnico completo para a equipe de desenvolvimento do Humanamente, garantindo que a interface PJ seja uma **tradução fiel** da experiência do Kimai, adaptada ao contexto e às necessidades específicas do projeto.

---

## 🔑 Principais Descobertas

### Estrutura de Navegação

-   **Header Global:** Timer ativo, botão de reiniciar, menu pessoal.
-   **Sidebar:** Menu expansível com seções para Dashboard, Timesheet, Calendário, Relatórios.
-   **Paleta de Cores:** Tema escuro com cores bem definidas para cada tipo de ação (verde para criar, azul para navegar, amarelo para filtrar, vermelho para alertas).

### Fluxo de Apontamento

1.  **Timer Ativo:** Usuário pode iniciar um timer sem preencher duração, e finalizá-lo depois.
2.  **Lançamento Manual:** Usuário pode criar registros completos (com data, hora de início e fim) diretamente.
3.  **Submissão para Aprovação:** Registros ficam em `draft` até que o usuário selecione múltiplos e clique em "Submeter para Aprovação", mudando o status para `pending`.

### Campos Obrigatórios

-   **Projeto:** Obrigatório, dropdown com busca.
-   **Atividade:** Obrigatório, dropdown filtrado pelo projeto.
-   **Data e Hora de Início:** Obrigatórios.
-   **Duração/Fim:** Opcional para timer ativo, obrigatório para registros completos.

### Validações

-   Não pode haver sobreposição de registros do mesmo usuário.
-   Duração não pode exceder limite configurado (ex: 12 horas).
-   Projeto e atividade devem estar ativos.

---

## 🚀 Próximos Passos

1.  **Revisar o documento mestre** (`analise_completa_kimai_pj.md`).
2.  **Mapear os componentes React** necessários para cada tela.
3.  **Criar as migrations do Supabase** para as tabelas de projetos, atividades, registros de tempo.
4.  **Implementar o fluxo de aprovação** com Edge Functions para notificações por e-mail.
5.  **Integrar com o módulo de pedidos de compra** (para PJ) após aprovação.

---

## 📧 Contato

Para dúvidas ou esclarecimentos sobre esta análise, entre em contato com a equipe de desenvolvimento do Humanamente.
