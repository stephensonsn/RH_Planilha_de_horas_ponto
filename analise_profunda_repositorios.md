# Análise Profunda: Kimai e MarcaPonto

**Autor:** Manus AI  
**Data:** 29 de Janeiro de 2026

---

## 1. KIMAI - Análise Completa

### 1.1. Estrutura de Dados (Entidade Timesheet)

A entidade `Timesheet` do Kimai possui a seguinte estrutura de campos:

| Campo | Tipo | Descrição | Obrigatório | Índices |
| :--- | :--- | :--- | :--- | :--- |
| `id` | INTEGER | ID único do registro | Sim | PRIMARY KEY |
| `date_tz` | DATE_IMMUTABLE | Data no timezone do usuário (para queries estatísticas) | Sim | `IDX_4F60C6B1BDF467148D93D649` |
| `start_time` | DATETIME | Data/hora de início | Sim | Múltiplos índices compostos |
| `end_time` | DATETIME | Data/hora de fim | Não | Múltiplos índices compostos |
| `timezone` | STRING(64) | Timezone do registro | Sim | - |
| `duration` | INTEGER | Duração em segundos | Não | `IDX_TIMESHEET_RESULT_STATS` |
| `break` | INTEGER | Tempo de intervalo em segundos | Não | - |
| `user` | FK → User | Usuário que criou o registro | Sim | Múltiplos índices |
| `activity_id` | FK → Activity | Atividade/tarefa | Sim | `IDX_4F60C6B181C06096` |
| `project_id` | FK → Project | Projeto | Sim | `IDX_TIMESHEET_RECENT_ACTIVITIES` |
| `description` | TEXT | Descrição do trabalho realizado | Não | - |
| `rate` | FLOAT | Valor total calculado | Sim | - |
| `internal_rate` | FLOAT | Taxa interna (custo) | Não | - |
| `fixed_rate` | FLOAT | Taxa fixa (se aplicável) | Não | - |
| `hourly_rate` | FLOAT | Taxa por hora | Não | - |
| `exported` | BOOLEAN | Se foi exportado (locked) | Sim | - |
| `billable` | BOOLEAN | Se é faturável | Sim | - |
| `category` | STRING(10) | Categoria (work, holiday, etc) | Sim | - |
| `tags` | ManyToMany → Tag | Tags associadas | Não | - |
| `meta` | OneToMany → TimesheetMeta | Campos customizados | Não | - |

### 1.2. Índices Estratégicos do Kimai

O Kimai utiliza **10 índices compostos** para otimizar queries:

```sql
-- Índice para buscar registros de um usuário
IDX_4F60C6B18D93D649 (user)

-- Índice para buscar por atividade
IDX_4F60C6B181C06096 (activity_id)

-- Índice para buscar registros de um usuário em um período
IDX_4F60C6B18D93D649502DF587 (user, start_time)

-- Índice para buscar registros por data de início
IDX_4F60C6B1502DF587 (start_time)

-- Índice para buscar registros em um intervalo de datas
IDX_4F60C6B1502DF58741561401 (start_time, end_time)

-- Índice para buscar registros de um usuário em um intervalo
IDX_4F60C6B1502DF587415614018D93D649 (start_time, end_time, user)

-- Índice para buscar registros finalizados de um usuário
IDX_4F60C6B1415614018D93D649 (end_time, user)

-- Índice para buscar registros por data no timezone do usuário
IDX_4F60C6B1BDF467148D93D649 (date_tz, user)

-- Índice para o modo "ticktac" (timer rápido)
IDX_TIMESHEET_TICKTAC (end_time, user, start_time)

-- Índice para atividades recentes do usuário
IDX_TIMESHEET_RECENT_ACTIVITIES (user, project_id, activity_id)

-- Índice para estatísticas e relatórios
IDX_TIMESHEET_RESULT_STATS (user, id, duration)
```

### 1.3. Regras de Negócio do Kimai

**Validações de Duração:**
- Duração máxima configurável (padrão: 8 horas)
- Não permite duração negativa
- Não permite `end_time` antes de `start_time`
- Não permite duração zero (exceto para deletar)

**Controle de Sobreposição:**
- Detecta registros sobrepostos do mesmo usuário
- Marca visualmente na interface com classe `.overlapping`
- Verificação baseada na ordenação (ASC ou DESC)

**Status de Exportação:**
- Registros exportados ficam bloqueados (locked)
- Apenas usuários com permissão `edit_exported_timesheet` podem editar
- Flag `exported` é permanente (não pode ser revertida facilmente)

**Cálculo de Taxas:**
- Taxa horária pode vir do projeto, atividade ou usuário (cascata)
- Taxa fixa sobrescreve o cálculo horário
- Taxa interna é separada da taxa de faturamento

**Billable (Faturável):**
- Modo automático (`BILLABLE_AUTOMATIC`): herda do projeto/atividade
- Modo manual: `BILLABLE_YES` ou `BILLABLE_NO`
- Registros não-faturáveis são excluídos de invoices

### 1.4. Fluxo de Telas do Kimai

**Tela Principal (`/timesheet`):**
- Lista de registros em formato de tabela (DataTable)
- Colunas: Data, Hora Início, Hora Fim, Duração, Intervalo, Taxa/Hora, Valor, Cliente, Projeto, Atividade, Descrição, Tags, Billable, Exported, Ações
- Resumo diário (summary row) agrupando por data
- Filtros: Usuário (apenas team), Intervalo de datas, Cliente, Projeto, Atividade, Tags, Status (running/stopped)
- Botão "+" para criar novo registro
- Botão de play na toolbar para iniciar timer
- Dropdown de "registros ativos" na toolbar

**Modal de Edição (`/timesheet/{id}/edit`):**
- Campos: Projeto, Atividade, Data, Hora Início, Hora Fim, Duração, Intervalo, Descrição, Tags
- Campos de taxa (se tiver permissão)
- Checkbox Billable
- Aviso se o registro estiver exportado (locked)
- Botões: Salvar, Cancelar, Deletar

**Modo Multi-Update (`/timesheet/multi-update`):**
- Seleção múltipla via checkboxes
- Atualização em lote de: Projeto, Atividade, Tags, Billable, Exported
- Confirmação antes de aplicar

**Exportação (`/timesheet/export/{exporter}`):**
- Filtros aplicados são mantidos
- Formatos: PDF, Excel, CSV, HTML
- Marca registros como exportados (locked)

---

## 2. MARCAPONTO - Análise Completa

### 2.1. Estrutura de Dados (Inferida do Código)

O MarcaPonto possui uma estrutura mais simples, focada em batida de ponto:

| Campo | Tipo | Descrição | Obrigatório |
| :--- | :--- | :--- | :--- |
| `id` | INTEGER | ID único do registro | Sim |
| `colaboradorId` | FK → Colaborador | Colaborador que bateu o ponto | Sim |
| `data` | DATE | Data da batida | Sim |
| `horario` | TIME | Hora da batida | Sim |
| `localizacao` | STRING | Coordenadas geográficas `[lat, long]` | Sim |
| `manual` | BOOLEAN | Se foi batida manual ou automática | Sim |
| `tipoDoRegistro` | ENUM | Tipo: Entrada, Saída, Intervalo Início, Intervalo Fim | Sim |
| `obs` | TEXT | Observações | Não |
| `status` | ENUM | Status: Pendente, Aprovado, Reprovado | Sim |
| `aprovadoPor` | FK → Gestor | Gestor que aprovou | Não |
| `dataAprovacao` | DATETIME | Data/hora da aprovação | Não |

### 2.2. Regras de Negócio do MarcaPonto

**Geolocalização Obrigatória:**
- Toda batida de ponto captura coordenadas GPS
- Validação de zona geográfica permitida (círculo com raio configurável)
- Múltiplas zonas podem ser associadas a um colaborador
- Batida fora da zona é bloqueada

**Tipos de Registro:**
- **Entrada**: Primeira batida do dia
- **Saída**: Última batida do dia
- **Intervalo Início**: Início do intervalo (almoço, etc)
- **Intervalo Fim**: Fim do intervalo

**Fluxo de Aprovação:**
- Todas as batidas ficam com status "Pendente"
- Gestor pode aprovar ou reprovar
- Reprovação exige justificativa (campo `obs`)
- Notificação em tempo real para o gestor quando há batidas pendentes

**Batida Manual:**
- Gestor pode criar batida manual para o colaborador
- Marcada com flag `manual: true`
- Exige justificativa no campo `obs`

### 2.3. Fluxo de Telas do MarcaPonto

**Tela de Batida de Ponto (Colaborador):**
- Relógio grande no centro
- Logo da empresa
- Botão "Marcar Ponto" (grande e destacado)
- Animação de loading durante captura de localização
- Modal de sucesso/erro após batida

**Tela de Espelho de Ponto (Colaborador):**
- Lista de batidas do mês
- Agrupamento por dia
- Exibição de: Data, Hora, Tipo, Status (Pendente/Aprovado/Reprovado)
- Filtro por mês

**Tela de Aprovação (Gestor):**
- Lista de batidas pendentes
- Filtros: Colaborador, Data, Tipo
- Card expandido com detalhes:
  - ID, Data, Horário, Tipo, Observação
  - Mapa com localização da batida
- Botões: Aprovar, Reprovar
- Notificação de contagem de pendências

**Tela de Relatórios (Gestor):**
- Relatório de batidas por colaborador
- Relatório de horas trabalhadas
- Relatório de ausências
- Exportação para PDF/Excel

**Tela de Gestão de Zonas (Gestor):**
- Mapa interativo (React Leaflet)
- Desenho de círculos (zonas permitidas)
- Associação de zonas a colaboradores
- Configuração de raio da zona

---

## 3. Mapeamento de Campos: Humanamente ↔ Kimai ↔ MarcaPonto

| Humanamente (Atual) | Kimai | MarcaPonto | Novo Campo Necessário |
| :--- | :--- | :--- | :--- |
| `id` | `id` | `id` | - |
| `tenant_id` | - | - | - |
| `user_id` | `user` | `colaboradorId` | - |
| `project_id` | `project_id` | - | - |
| `entry_date` | `date_tz` | `data` | - |
| `clock_in` | `start_time` | `horario` (Entrada) | - |
| `clock_out` | `end_time` | `horario` (Saída) | - |
| `break_start` | - | `horario` (Intervalo Início) | **Adicionar** |
| `break_end` | - | `horario` (Intervalo Fim) | **Adicionar** |
| `break_minutes` | `break` | - | - |
| `total_minutes` | `duration` | - | - |
| `overtime_minutes` | - | - | - |
| `night_shift_minutes` | - | - | - |
| `notes` | `description` | `obs` | - |
| `task_description` | - | - | - |
| `status` | - | `status` | - |
| `approved_by` | - | `aprovadoPor` | - |
| `approved_at` | - | `dataAprovacao` | - |
| `rejection_reason` | - | - | **Adicionar** |
| `location` | - | `localizacao` | **Expandir para lat/long** |
| `is_remote` | - | - | - |
| - | `timezone` | - | **Adicionar** |
| - | `rate` | - | - |
| - | `hourly_rate` | - | - |
| - | `fixed_rate` | - | - |
| - | `internal_rate` | - | - |
| - | `exported` | - | - |
| - | `billable` | - | - |
| - | `category` | - | - |
| - | `tags` | - | - |
| - | - | `manual` | **Adicionar** |
| - | - | `tipoDoRegistro` | **Adicionar** |
| - | `activity_id` | - | **Adicionar** |

---

## 4. Novos Campos e Tabelas Necessários

### 4.1. Alterações na Tabela `profiles`

```sql
ALTER TABLE public.profiles
ADD COLUMN contract_type TEXT DEFAULT 'CLT' CHECK (contract_type IN ('CLT', 'PJ')),
ADD COLUMN hourly_rate NUMERIC(10, 2) DEFAULT 0.00;

CREATE INDEX idx_profiles_contract_type ON profiles(contract_type);
```

### 4.2. Alterações na Tabela `timesheet_entries`

```sql
ALTER TABLE public.timesheet_entries
ADD COLUMN clock_in_latitude NUMERIC(10, 8),
ADD COLUMN clock_in_longitude NUMERIC(11, 8),
ADD COLUMN clock_in_address TEXT,
ADD COLUMN clock_out_latitude NUMERIC(10, 8),
ADD COLUMN clock_out_longitude NUMERIC(11, 8),
ADD COLUMN clock_out_address TEXT,
ADD COLUMN device_info TEXT,
ADD COLUMN timezone TEXT DEFAULT 'America/Sao_Paulo',
ADD COLUMN activity_id UUID REFERENCES timesheet_activities(id),
ADD COLUMN is_manual BOOLEAN DEFAULT false,
ADD COLUMN entry_type TEXT CHECK (entry_type IN ('clock_in', 'clock_out', 'break_start', 'break_end', 'manual')),
ADD COLUMN rejection_reason TEXT;

CREATE INDEX idx_timesheet_entries_user_date ON timesheet_entries(user_id, entry_date);
CREATE INDEX idx_timesheet_entries_geolocation ON timesheet_entries(clock_in_latitude, clock_in_longitude);
CREATE INDEX idx_timesheet_entries_status ON timesheet_entries(status);
CREATE INDEX idx_timesheet_entries_activity ON timesheet_entries(activity_id);
```

### 4.3. Nova Tabela `timesheet_activities`

```sql
CREATE TABLE public.timesheet_activities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  project_id UUID REFERENCES timesheet_projects(id) ON DELETE CASCADE,
  code TEXT,
  name TEXT NOT NULL,
  description TEXT,
  is_active BOOLEAN DEFAULT true,
  color TEXT DEFAULT '#10b981',
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_timesheet_activities_tenant ON timesheet_activities(tenant_id);
CREATE INDEX idx_timesheet_activities_project ON timesheet_activities(project_id);

ALTER TABLE timesheet_activities ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Tenant users can view activities"
  ON timesheet_activities FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM profiles
      WHERE profiles.id = auth.uid()
      AND profiles.tenant_id = timesheet_activities.tenant_id
    )
  );

CREATE POLICY "Admins can manage activities"
  ON timesheet_activities FOR ALL
  USING (
    EXISTS (
      SELECT 1 FROM profiles
      WHERE profiles.id = auth.uid()
      AND profiles.tenant_id = timesheet_activities.tenant_id
      AND profiles.role IN ('admin', 'gestor')
    )
  );
```

### 4.4. Nova Tabela `geolocation_zones`

```sql
CREATE TABLE public.geolocation_zones (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_id UUID,
  name TEXT NOT NULL,
  center_latitude NUMERIC(10, 8) NOT NULL,
  center_longitude NUMERIC(11, 8) NOT NULL,
  radius_meters INTEGER DEFAULT 100,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_geolocation_zones_tenant ON geolocation_zones(tenant_id);
CREATE INDEX idx_geolocation_zones_user ON geolocation_zones(user_id);

ALTER TABLE geolocation_zones ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Admins can manage geolocation zones"
  ON geolocation_zones FOR ALL
  USING (
    EXISTS (
      SELECT 1 FROM profiles
      WHERE profiles.id = auth.uid()
      AND profiles.tenant_id = geolocation_zones.tenant_id
      AND profiles.role IN ('admin', 'gestor')
    )
  );
```

### 4.5. Nova Tabela `purchase_orders`

```sql
CREATE TABLE public.purchase_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_id UUID NOT NULL,
  timesheet_summary_id UUID REFERENCES timesheet_monthly_summary(id),
  status TEXT DEFAULT 'pending_invoice' CHECK (status IN ('pending_invoice', 'invoiced', 'cancelled')),
  total_amount NUMERIC(10, 2) NOT NULL,
  total_hours NUMERIC(10, 2) NOT NULL,
  hourly_rate NUMERIC(10, 2) NOT NULL,
  period_start DATE NOT NULL,
  period_end DATE NOT NULL,
  issue_date DATE NOT NULL,
  invoice_number TEXT,
  invoice_file_url TEXT,
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_purchase_orders_tenant_user ON purchase_orders(tenant_id, user_id);
CREATE INDEX idx_purchase_orders_status ON purchase_orders(status);
CREATE INDEX idx_purchase_orders_period ON purchase_orders(period_start, period_end);

ALTER TABLE purchase_orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Admins can manage purchase orders"
  ON purchase_orders FOR ALL
  USING (
    EXISTS (
      SELECT 1 FROM profiles
      WHERE profiles.id = auth.uid()
      AND profiles.tenant_id = purchase_orders.tenant_id
      AND profiles.role IN ('admin', 'gestor')
    )
  );

CREATE POLICY "Users can view own purchase orders"
  ON purchase_orders FOR SELECT
  USING (user_id = auth.uid());
```

---

Este documento será continuado com wireframes, validações e túneis de fluxo na próxima seção.
