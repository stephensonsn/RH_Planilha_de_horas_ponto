# Documentação de Validações e Túneis de Fluxo

**Autor:** Manus AI  
**Data:** 29 de Janeiro de 2026

---

## 1. VALIDAÇÕES DO LADO DO CLIENTE (FRONTEND)

### Formulário de Apontamento de Horas (PJ)

- **Campo `Projeto`:** Obrigatório. O dropdown não deve ter uma opção nula selecionável após o primeiro carregamento.
- **Campo `Atividade`:** Obrigatório.
- **Campo `Início` (Data/Hora):** Obrigatório. Não pode ser uma data/hora futura. Não pode ser anterior à data de início do projeto.
- **Campo `Fim` (Data/Hora):** Não pode ser anterior ao campo `Início`. A duração máxima entre `Início` e `Fim` é de 12 horas (configurável pelo tenant).
- **Campo `Duração`:** Se o `Fim` for preenchido, a duração é calculada automaticamente. Se a duração for inserida manualmente, o campo `Fim` é que é atualizado. Formatos aceitos: `HH:mm` (ex: `02:30`), decimal (ex: `2.5`), e descritivo (ex: `2h 30m`).

### Formulário de Batida de Ponto (CLT)

- **Botão `Marcar Ponto`:** Desabilitado se a geolocalização não estiver disponível ou se o usuário não der permissão. Uma mensagem clara deve instruir o usuário a habilitar o serviço de localização.

---

## 2. VALIDAÇÕES DO LADO DO SERVIDOR (BACKEND - SUPABASE)

### Tabela `timesheet_entries` (RLS - Row Level Security)

- **`SELECT`:** Usuários só podem ver seus próprios registros. Gestores (`role = 'manager'`) podem ver os registros de todos os usuários do seu `tenant_id`.
- **`INSERT`:** Usuários só podem inserir registros para si mesmos (`user_id = auth.uid()`).
- **`UPDATE`:** Usuários só podem editar seus próprios registros, e apenas se o status for `draft` ou `pending`. Gestores podem editar qualquer registro do seu tenant, exceto os que já foram faturados (`status = 'invoiced'`).
- **`DELETE`:** Usuários só podem deletar seus próprios registros se o status for `draft`.

### Funções de Validação (PostgreSQL Functions)

- **`function validate_geolocation()`:**
  - **Input:** `p_user_id`, `p_tenant_id`, `p_latitude`, `p_longitude`.
  - **Lógica:** Busca na tabela `geolocation_zones` por zonas ativas para o `user_id` ou para o `tenant_id` (zonas globais). Usa a extensão `postgis` para calcular se o ponto (`p_latitude`, `p_longitude`) está dentro de algum dos polígonos (`zone_polygon`).
  - **Retorno:** `boolean` (true se estiver dentro, false se não).

- **`function check_overlapping_entries()` (Trigger):**
  - **Gatilho:** `BEFORE INSERT OR UPDATE` na tabela `timesheet_entries`.
  - **Lógica:** Verifica se o novo intervalo de tempo (`clock_in`, `clock_out`) se sobrepõe a qualquer outro registro existente para o mesmo `user_id`.
  - **Ação:** Se houver sobreposição, levanta uma exceção (`RAISE EXCEPTION 'Time entry overlaps with an existing one.'`), impedindo a inserção/atualização.

---

## 3. TÚNEIS DE FLUXO (USER FLOWS)

### Fluxo 1: Onboarding de Novo Colaborador (PJ)

1.  **Gestor:** Acessa `Gestão > Usuários` e clica em `Novo Usuário`.
2.  **Sistema:** Exibe formulário com campos: Nome, Email, Cargo, `contract_type` (seleciona "PJ"), `hourly_rate` (valor/hora), e projetos associados.
3.  **Gestor:** Preenche e salva.
4.  **Sistema (Trigger/Edge Function):**
    - Cria o usuário no `auth.users`.
    - Insere um novo perfil na tabela `profiles` com os dados fornecidos.
    - Envia um e-mail de boas-vindas (via Resend) com um link para definição de senha.
5.  **Colaborador:** Clica no link, define sua senha e é redirecionado para o dashboard.
6.  **Sistema:** O dashboard do colaborador PJ exibe a interface inspirada no Kimai (tabela de apontamentos, timer, etc.).

### Fluxo 2: Batida de Ponto com Restrição Geográfica (CLT)

1.  **Gestor:** Acessa `Gestão > Usuários`, seleciona um colaborador CLT e vai para a aba `Zonas de Ponto`.
2.  **Sistema:** Exibe um mapa (Leaflet).
3.  **Gestor:** Desenha um polígono no mapa, definindo a área onde o ponto é permitido, e salva.
4.  **Sistema:** Salva as coordenadas do polígono na tabela `geolocation_zones`, associado ao `user_id`.
5.  **Colaborador:** Abre o app Humanamente e acessa a tela de "Bater Ponto".
6.  **Sistema:** Solicita permissão de geolocalização. Exibe o `ClockInCard`.
7.  **Colaborador:** Clica em `Marcar Ponto`.
8.  **Sistema (Frontend):** Captura as coordenadas `lat/lng`.
9.  **Sistema (Backend):**
    - Chama a função `validate_geolocation()` com as coordenadas.
    - **Se `true`:** Insere o registro na `timesheet_entries` e exibe mensagem de sucesso.
    - **Se `false`:** Retorna um erro e o frontend exibe a mensagem: "Você está fora da área permitida para bater o ponto."

### Fluxo 3: Aprovação de Horas e Faturamento (PJ)

1.  **Colaborador:** Na sua tela de apontamentos, seleciona os registros da semana/mês e clica em `Submeter para Aprovação`.
2.  **Sistema:** Muda o status dos registros selecionados de `draft` para `pending`.
3.  **Sistema (Edge Function `on_pending_entry`):** Detecta a mudança, encontra o gestor do usuário e dispara um e-mail: "Você tem horas de [Nome do Colaborador] para aprovar."
4.  **Gestor:** Acessa a tela `Aprovações`, vê os registros pendentes, e clica em `Aprovar`.
5.  **Sistema:** Muda o status dos registros para `approved`.
6.  **Sistema (Edge Function `on_approved_pj_entry`):**
    - Detecta a mudança para `approved` em um registro PJ.
    - Agrega o valor total (`total_value`) de todos os registros aprovados para aquele colaborador no período.
    - Cria uma nova linha na tabela `purchase_orders` com o valor total, `user_id`, e `status = 'pending_invoice'`.
7.  **Financeiro/Gestor:** Na tela `Financeiro > Pedidos de Compra`, vê o novo pedido e clica em `Solicitar Emissão de NF`.
8.  **Sistema (Edge Function `on_request_invoice`):** Dispara um e-mail para o colaborador: "Por favor, emita a Nota Fiscal no valor de R$ [valor] e faça o upload no portal."

---
