# Módulo de Timesheet e Aprovações - Humanamente

**Projeto:** Substituição do Módulo de Horas e Batida de Ponto  
**Stack:** React + TypeScript + Supabase + Tailwind CSS  
**Baseado em:** Análise Interativa das Demos Kimai e MarcaPonto

---

## 📋 Índice de Documentação

### 🎯 **DOCUMENTO MESTRE**
👉 **[ARQUITETURA_COMPLETA_TIMESHEET_HUMANAMENTE.md](./ARQUITETURA_COMPLETA_TIMESHEET_HUMANAMENTE.md)** 👈

Este é o **documento principal** que consolida toda a arquitetura visual, funcional e técnica do módulo. Ele unifica as análises dos perfis **Colaborador (PJ e CLT)** e **Team Leader (Gestor)**, fornecendo um guia único e completo para a implementação.

**Conteúdo:**
- ✅ Wireframes completos de todas as interfaces (PJ, CLT, Gestor)
- ✅ Arquitetura técnica do banco de dados (Supabase)
- ✅ Componentes React principais
- ✅ Automações (Edge Functions)
- ✅ Fluxo de dados e regras de negócio
- ✅ Checklist de implementação completo

---

### 📂 Documentação Detalhada por Perfil

#### 1. Análise do Colaborador PJ (Kimai)
📁 **[analise_kimai_detalhada/](./analise_kimai_detalhada/)**

Análise completa da interface de apontamento de horas para terceiros (PJ), baseada na demo interativa do Kimai.

**Documentos:**
- `analise_completa_kimai_pj.md` - Documento mestre com wireframes e fluxos
- `analise_kimai_dashboard.md` - Dashboard e widgets
- `analise_kimai_timesheet.md` - Tela de apontamentos
- `analise_kimai_modal_criar.md` - Modal de criação/edição
- `analise_kimai_calendario.md` - Visualização em calendário

#### 2. Análise do Team Leader (Gestor)
📁 **[analise_teamlead/](./analise_teamlead/)**

Análise completa da interface de aprovação de horas para gestores, baseada na demo interativa do Kimai.

**Documentos:**
- `analise_completa_teamlead.md` - Documento mestre com wireframes e fluxos
- `analise_teamlead_dashboard.md` - Dashboard do gestor
- `analise_teamlead_all_times.md` - Tela de aprovação (central de comando)

#### 3. Documentação Técnica Original
📁 **[Raiz do Repositório](.)**

Documentos técnicos criados na primeira fase da análise:

- `PLANO_COMPLETO_INTEGRACAO_TIMESHEET.md` - Plano estratégico inicial
- `analise_profunda_repositorios.md` - Análise do código-fonte do Kimai e MarcaPonto
- `wireframes_e_fluxos.md` - Wireframes ASCII art e fluxos de tela
- `arquitetura_banco_dados.md` - Schema completo com migrations SQL
- `traducoes_regras_negocio.md` - Componentes React traduzidos
- `validacoes_tuneis_fluxo.md` - Validações e túneis de fluxo

---

## 🎯 Visão Geral da Solução

### Diferenciação de Perfis

**Colaborador PJ (Kimai):**
- Interface rica com timer ativo e lançamento manual
- Apontamento por projeto e atividade
- Cálculo automático de valor (horas × valor/hora)
- Widget de valor projetado com toggle de visibilidade

**Colaborador CLT (MarcaPonto):**
- Interface simplificada de batida de ponto
- Captura automática de geolocalização
- Espelho de ponto do dia
- Sem valores financeiros na home

**Team Leader (Gestor):**
- Visão consolidada de toda a equipe
- Filtros avançados (status, usuário, período, projeto)
- Ações em massa (aprovar, rejeitar, exportar)
- Modal de rejeição com motivo obrigatório

### Fluxo End-to-End

1. **Apontamento:** Colaboradores registram suas horas (PJ) ou batem ponto (CLT)
2. **Submissão:** Ao final do período, submetem para aprovação (status: `pending`)
3. **Notificação:** Gestor é notificado por e-mail
4. **Aprovação:** Gestor aprova ou rejeita os registros
5. **Automação:**
   - Se aprovado (PJ): Cria Pedido de Compra automaticamente
   - Se rejeitado: Notifica colaborador com motivo
   - E-mails disparados em cada etapa

---

## 🔧 Stack Tecnológica

- **Frontend:** React + TypeScript + Tailwind CSS
- **Backend:** Supabase (PostgreSQL + Edge Functions)
- **Autenticação:** Supabase Auth
- **Geolocalização:** PostGIS (extensão do PostgreSQL)
- **E-mails:** Resend ou SendGrid

---

## 📦 Estrutura do Repositório

```
RH_Planilha_de_horas_ponto/
├── README.md (este arquivo)
├── ARQUITETURA_COMPLETA_TIMESHEET_HUMANAMENTE.md ⭐ DOCUMENTO MESTRE
├── PLANO_COMPLETO_INTEGRACAO_TIMESHEET.md
├── analise_profunda_repositorios.md
├── wireframes_e_fluxos.md
├── arquitetura_banco_dados.md
├── traducoes_regras_negocio.md
├── validacoes_tuneis_fluxo.md
├── analise_kimai_detalhada/
│   ├── README.md
│   ├── analise_completa_kimai_pj.md
│   ├── analise_kimai_dashboard.md
│   ├── analise_kimai_timesheet.md
│   ├── analise_kimai_modal_criar.md
│   └── analise_kimai_calendario.md
├── analise_teamlead/
│   ├── README.md
│   ├── analise_completa_teamlead.md
│   ├── analise_teamlead_dashboard.md
│   └── analise_teamlead_all_times.md
└── assets/
    └── (20 screenshots de referência)
```

---

## 🚀 Como Usar Esta Documentação

1. **Comece pelo Documento Mestre:** Leia `ARQUITETURA_COMPLETA_TIMESHEET_HUMANAMENTE.md` para ter uma visão geral completa.
2. **Aprofunde-se nos Perfis:** Consulte as pastas `analise_kimai_detalhada/` e `analise_teamlead/` para detalhes específicos de cada interface.
3. **Implemente Fase por Fase:** Siga o checklist de implementação no documento mestre.
4. **Consulte os Wireframes:** Use os wireframes ASCII art como guia visual durante o desenvolvimento.

---

## 📧 Contato

Para dúvidas ou esclarecimentos sobre esta documentação, entre em contato com a equipe de desenvolvimento do Humanamente.

---

**Autor:** Manus AI  
**Data:** 01 de Fevereiro de 2026
