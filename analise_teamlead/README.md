# Análise Completa: Interface do Team Leader (Gestor) - Kimai

**Projeto:** Humanamente - Módulo de Aprovação de Horas  
**Baseado em:** Demo Interativa Kimai (https://demo-empty.kimai.org)  
**Credenciais de Acesso:** `tony_teamlead` / `password`  
**Data da Análise:** 01 de Fevereiro de 2026

---

## 📋 Conteúdo do Diretório

Este diretório contém a análise completa da interface do **Team Leader** (Gestor) no Kimai, capturada através de interação direta com a demonstração ao vivo. Esta análise complementa a documentação do colaborador PJ, completando o ciclo de apontamento → submissão → aprovação.

### Documentos Incluídos:

1.  **`analise_completa_teamlead.md`** - **DOCUMENTO MESTRE**
    -   Sumário executivo do fluxo end-to-end (colaborador → gestor).
    -   Wireframe adaptado da tela de aprovação para o Humanamente.
    -   Detalhamento completo do fluxo de aprovação (aprovar, rejeitar, editar).
    -   Integrações necessárias com a documentação existente (banco de dados, componentes, Edge Functions).
    -   Checklist de implementação específico para o perfil de gestor.

2.  **`analise_teamlead_dashboard.md`**
    -   Análise detalhada do Dashboard do Team Leader.
    -   Diferenças estruturais em relação ao Dashboard do colaborador.
    -   Menu expandido com funcionalidades exclusivas (Export, All times, Invoices, Administration).
    -   Widgets e informações exibidas.

3.  **`analise_teamlead_all_times.md`**
    -   Análise profunda da tela **"All times"** (central de aprovação).
    -   Wireframe ASCII da estrutura da tela.
    -   Descrição completa da barra de ferramentas (filtros, busca, ações).
    -   Estrutura da tabela de registros (colunas, status, ações).
    -   Fluxos de aprovação individual e em massa.
    -   Fluxos de rejeição e edição.
    -   Filtros avançados disponíveis.
    -   Ações em massa (aprovar, rejeitar, deletar, exportar).
    -   Botão "For multiple users" (criação em massa).
    -   Exportação de dados (PDF, Excel, CSV, HTML).
    -   Diferenças em relação à tela "My times" do colaborador.
    -   Regras de negócio (permissões, validações, notificações).
    -   Wireframe adaptado para o Humanamente (com valores financeiros).

---

## 🎯 Objetivo da Análise

Fornecer um guia técnico completo para a equipe de desenvolvimento do Humanamente, garantindo que a **interface de aprovação do gestor** seja uma tradução fiel da experiência do Kimai, com adaptações específicas para o contexto do projeto (diferenciação PJ/CLT, integração com pedidos de compra, etc.).

---

## 🔑 Principais Descobertas

### Estrutura de Navegação (Gestor)

O Team Leader tem acesso a **funcionalidades exclusivas** no menu lateral:
- **Export:** Exportar dados de toda a equipe.
- **All times:** Visualizar e aprovar horas de todos os colaboradores (central de aprovação).
- **Invoices:** Gestão de faturas e cobranças.
- **Administration:** Configurações de projetos, usuários, clientes, etc.

### Fluxo de Aprovação (3 Cenários)

1.  **Aprovação Individual:** Gestor clica no ícone de "Approve" (✓) em um registro específico.
2.  **Aprovação em Massa:** Gestor seleciona múltiplos registros via checkbox e clica em "Aprovar Selecionados".
3.  **Rejeição:** Gestor clica no ícone de "Reject" (✗), insere um motivo obrigatório e confirma.

### Automações Necessárias (Supabase Edge Functions)

1.  **`on-timesheet-submitted`:** Quando o colaborador submete horas (`pending`), envia e-mail para o Team Leader.
2.  **`on-timesheet-approved`:** Quando o gestor aprova:
    -   Cria Pedido de Compra (se PJ).
    -   Envia e-mail para o colaborador e para o financeiro.
3.  **`on-timesheet-rejected`:** Quando o gestor rejeita:
    -   Envia e-mail para o colaborador com o motivo da rejeição.

### Campos Obrigatórios da Tela de Aprovação

- **User:** Nome do colaborador (filtro disponível).
- **Date:** Data do registro.
- **From / To / Duration:** Horário e duração.
- **Project / Activity:** Projeto e atividade associados.
- **Status:** Draft, Pending, Approved, Rejected (filtro principal).
- **Actions:** Ícones de aprovar, rejeitar e editar.

### Validações Implementadas

- Não é possível aprovar registros com **sobreposição de horários** do mesmo usuário.
- Não é possível aprovar registros com **duração zero**.
- Não é possível aprovar registros de **projetos inativos**.
- O motivo da rejeição é **obrigatório**.

---

## 🚀 Próximos Passos

1.  **Revisar o documento mestre** (`analise_completa_teamlead.md`).
2.  **Atualizar a arquitetura do banco de dados** com as novas tabelas e campos (status, rejection_reason, purchase_orders, timesheet_edits).
3.  **Desenvolver o componente React `ApprovalTimesheetTable.tsx`** com filtros, ações em massa e modais de rejeição/edição.
4.  **Implementar as 3 Edge Functions** do Supabase para automação de e-mails e pedidos de compra.
5.  **Integrar com o sistema de permissões** (RLS) para que gestores só vejam suas equipes.
6.  **Testar o fluxo end-to-end** (colaborador aponta → submete → gestor aprova → pedido de compra criado → e-mails enviados).

---

## 📧 Contato

Para dúvidas ou esclarecimentos sobre esta análise, entre em contato com a equipe de desenvolvimento do Humanamente.
