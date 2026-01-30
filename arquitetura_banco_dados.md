# Arquitetura Completa do Banco de Dados: Humanamente Timesheet

**Autor:** Manus AI  
**Data:** 29 de Janeiro de 2026

---

## 1. DIAGRAMA ER (Entity Relationship)

```
┌──────────────┐         ┌──────────────────┐         ┌──────────────┐
│   tenants    │────────▶│    profiles      │◀────────│     users    │
│              │ 1     * │                  │ 1     1 │              │
└──────────────┘         │ - contract_type  │         └──────────────┘
                         │ - hourly_rate    │
                         └────────┬─────────┘
                                  │ 1
                                  │
                                  │ *
                         ┌────────▼─────────┐
                         │ timesheet_entries│
                         │                  │
                         │ - entry_type     │
                         │ - clock_in       │
                         │ - clock_out      │
                         │ - break_start    │
                         │ - break_end      │
                         │ - location_*     │
                         │ - is_manual      │
                         │ - status         │
                         └────────┬─────────┘
                                  │ *
                    ┌─────────────┼─────────────┐
                    │             │             │
                    │ *           │ *           │ *
         ┌──────────▼────────┐   │   ┌────────▼────────┐
         │ timesheet_projects│   │   │timesheet_       │
         │                   │   │   │activities       │
         │ - hourly_rate     │   │   │                 │
         │ - budget_hours    │   │   │ - project_id    │
         └───────────────────┘   │   └─────────────────┘
                                 │
                                 │ *
                         ┌───────▼──────────┐
                         │ geolocation_zones│
                         │                  │
                         │ - center_lat     │
                         │ - center_long    │
                         │ - radius_meters  │
                         └──────────────────┘

         ┌──────────────────┐         ┌──────────────────┐
         │ purchase_orders  │◀────────│timesheet_monthly_│
         │                  │ 1     1 │summary           │
         │ - status         │         │                  │
         │ - total_amount   │         │ - total_hours    │
         │ - invoice_number │         │ - total_value    │
         └──────────────────┘         └──────────────────┘
```

---

## 2. MIGRATIONS COMPLETAS (Supabase SQL)

### Migration 1: Alterações na Tabela `profiles`

```sql
-- Migration: 20260129_001_add_contract_type_to_profiles
-- Description: Adiciona campos para diferenciar CLT e PJ

ALTER TABLE public.profiles
ADD COLUMN IF NOT EXISTS contract_type TEXT DEFAULT 'CLT' CHECK (contract_type IN ('CLT', 'PJ')),
ADD COLUMN IF NOT EXISTS hourly_rate NUMERIC(10, 2) DEFAULT 0.00,
ADD COLUMN IF NOT EXISTS show_earnings BOOLEAN DEFAULT false;

-- Índice para otimizar queries por tipo de contrato
CREATE INDEX IF NOT EXISTS idx_profiles_contract_type 
ON profiles(contract_type);

-- Índice composto para queries de relatórios
CREATE INDEX IF NOT EXISTS idx_profiles_tenant_contract 
ON profiles(tenant_id, contract_type);

COMMENT ON COLUMN profiles.contract_type IS 'Tipo de contrato: CLT (batida de ponto) ou PJ (apontamento de horas)';
COMMENT ON COLUMN profiles.hourly_rate IS 'Taxa horária padrão do colaborador PJ';
COMMENT ON COLUMN profiles.show_earnings IS 'Se o colaborador quer visualizar valores financeiros na home';
```

### Migration 2: Expansão da Tabela `timesheet_entries`

```sql
-- Migration: 20260129_002_expand_timesheet_entries
-- Description: Adiciona campos para geolocalização, tipos de batida e aprovação

ALTER TABLE public.timesheet_entries
-- Geolocalização de entrada
ADD COLUMN IF NOT EXISTS clock_in_latitude NUMERIC(10, 8),
ADD COLUMN IF NOT EXISTS clock_in_longitude NUMERIC(11, 8),
ADD COLUMN IF NOT EXISTS clock_in_address TEXT,

-- Geolocalização de saída
ADD COLUMN IF NOT EXISTS clock_out_latitude NUMERIC(10, 8),
ADD COLUMN IF NOT EXISTS clock_out_longitude NUMERIC(11, 8),
ADD COLUMN IF NOT EXISTS clock_out_address TEXT,

-- Informações do dispositivo (para auditoria)
ADD COLUMN IF NOT EXISTS device_info JSONB,

-- Timezone do registro
ADD COLUMN IF NOT EXISTS timezone TEXT DEFAULT 'America/Sao_Paulo',

-- Atividade associada (para PJ)
ADD COLUMN IF NOT EXISTS activity_id UUID REFERENCES timesheet_activities(id) ON DELETE SET NULL,

-- Tipo de entrada (para CLT)
ADD COLUMN IF NOT EXISTS entry_type TEXT CHECK (entry_type IN ('clock_in', 'clock_out', 'break_start', 'break_end', 'manual')),

-- Se foi batida manual (gestor criou para o colaborador)
ADD COLUMN IF NOT EXISTS is_manual BOOLEAN DEFAULT false,

-- Intervalo (para CLT)
ADD COLUMN IF NOT EXISTS break_start TIME,
ADD COLUMN IF NOT EXISTS break_end TIME,

-- Motivo da rejeição
ADD COLUMN IF NOT EXISTS rejection_reason TEXT,

-- Quem aprovou/rejeitou
ADD COLUMN IF NOT EXISTS approved_by UUID REFERENCES profiles(id) ON DELETE SET NULL,

-- Campos de taxa e valor (para PJ)
ADD COLUMN IF NOT EXISTS hourly_rate NUMERIC(10, 2),
ADD COLUMN IF NOT EXISTS total_value NUMERIC(10, 2),
ADD COLUMN IF NOT EXISTS is_billable BOOLEAN DEFAULT true;

-- Índices estratégicos (baseados no Kimai)
CREATE INDEX IF NOT EXISTS idx_timesheet_entries_user_date 
ON timesheet_entries(user_id, entry_date);

CREATE INDEX IF NOT EXISTS idx_timesheet_entries_user_clock_in 
ON timesheet_entries(user_id, clock_in);

CREATE INDEX IF NOT EXISTS idx_timesheet_entries_clock_in_out 
ON timesheet_entries(clock_in, clock_out);

CREATE INDEX IF NOT EXISTS idx_timesheet_entries_geolocation 
ON timesheet_entries(clock_in_latitude, clock_in_longitude);

CREATE INDEX IF NOT EXISTS idx_timesheet_entries_status 
ON timesheet_entries(status);

CREATE INDEX IF NOT EXISTS idx_timesheet_entries_activity 
ON timesheet_entries(activity_id);

CREATE INDEX IF NOT EXISTS idx_timesheet_entries_project 
ON timesheet_entries(project_id);

CREATE INDEX IF NOT EXISTS idx_timesheet_entries_tenant_user_date 
ON timesheet_entries(tenant_id, user_id, entry_date);

-- Índice para detectar registros ativos (timer rodando)
CREATE INDEX IF NOT EXISTS idx_timesheet_entries_active 
ON timesheet_entries(user_id, clock_out) 
WHERE clock_out IS NULL;

-- Índice para estatísticas e relatórios
CREATE INDEX IF NOT EXISTS idx_timesheet_entries_stats 
ON timesheet_entries(user_id, id, total_minutes);

COMMENT ON COLUMN timesheet_entries.entry_type IS 'Tipo de batida para CLT: clock_in, clock_out, break_start, break_end, manual';
COMMENT ON COLUMN timesheet_entries.is_manual IS 'Se a batida foi criada manualmente por um gestor';
COMMENT ON COLUMN timesheet_entries.device_info IS 'Informações do dispositivo usado na batida (browser, OS, IP)';
```

### Migration 3: Tabela `timesheet_activities`

```sql
-- Migration: 20260129_003_create_timesheet_activities
-- Description: Cria tabela de atividades/tarefas para PJ

CREATE TABLE IF NOT EXISTS public.timesheet_activities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  project_id UUID REFERENCES timesheet_projects(id) ON DELETE CASCADE,
  code TEXT,
  name TEXT NOT NULL,
  description TEXT,
  is_active BOOLEAN DEFAULT true,
  color TEXT DEFAULT '#10b981',
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  
  CONSTRAINT timesheet_activities_name_check CHECK (char_length(name) >= 2),
  CONSTRAINT timesheet_activities_code_check CHECK (code IS NULL OR char_length(code) >= 2)
);

-- Índices
CREATE INDEX idx_timesheet_activities_tenant 
ON timesheet_activities(tenant_id);

CREATE INDEX idx_timesheet_activities_project 
ON timesheet_activities(project_id);

CREATE INDEX idx_timesheet_activities_active 
ON timesheet_activities(is_active) 
WHERE is_active = true;

-- RLS Policies
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

-- Trigger para updated_at
CREATE TRIGGER set_timesheet_activities_updated_at
  BEFORE UPDATE ON timesheet_activities
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

COMMENT ON TABLE timesheet_activities IS 'Atividades/tarefas que podem ser associadas a projetos para apontamento de horas PJ';
```

### Migration 4: Tabela `geolocation_zones`

```sql
-- Migration: 20260129_004_create_geolocation_zones
-- Description: Cria tabela de zonas geográficas permitidas para batida de ponto CLT

CREATE TABLE IF NOT EXISTS public.geolocation_zones (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  center_latitude NUMERIC(10, 8) NOT NULL,
  center_longitude NUMERIC(11, 8) NOT NULL,
  radius_meters INTEGER DEFAULT 100 CHECK (radius_meters > 0 AND radius_meters <= 10000),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  
  CONSTRAINT geolocation_zones_name_check CHECK (char_length(name) >= 2),
  CONSTRAINT geolocation_zones_latitude_check CHECK (center_latitude BETWEEN -90 AND 90),
  CONSTRAINT geolocation_zones_longitude_check CHECK (center_longitude BETWEEN -180 AND 180)
);

-- Índices
CREATE INDEX idx_geolocation_zones_tenant 
ON geolocation_zones(tenant_id);

CREATE INDEX idx_geolocation_zones_user 
ON geolocation_zones(user_id);

CREATE INDEX idx_geolocation_zones_active 
ON geolocation_zones(is_active) 
WHERE is_active = true;

-- Índice espacial para queries de proximidade
CREATE INDEX idx_geolocation_zones_coordinates 
ON geolocation_zones(center_latitude, center_longitude);

-- RLS Policies
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

CREATE POLICY "Users can view own zones"
  ON geolocation_zones FOR SELECT
  USING (
    user_id = auth.uid() OR user_id IS NULL
  );

-- Trigger para updated_at
CREATE TRIGGER set_geolocation_zones_updated_at
  BEFORE UPDATE ON geolocation_zones
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

COMMENT ON TABLE geolocation_zones IS 'Zonas geográficas permitidas para batida de ponto CLT';
COMMENT ON COLUMN geolocation_zones.user_id IS 'Se NULL, a zona é global para todos os colaboradores do tenant';
COMMENT ON COLUMN geolocation_zones.radius_meters IS 'Raio da zona em metros (máximo 10km)';
```

### Migration 5: Tabela `purchase_orders`

```sql
-- Migration: 20260129_005_create_purchase_orders
-- Description: Cria tabela de pedidos de compra gerados após aprovação de horas PJ

CREATE TABLE IF NOT EXISTS public.purchase_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  timesheet_summary_id UUID REFERENCES timesheet_monthly_summary(id) ON DELETE SET NULL,
  
  status TEXT DEFAULT 'pending_invoice' CHECK (status IN ('pending_invoice', 'invoiced', 'cancelled')),
  
  total_amount NUMERIC(10, 2) NOT NULL CHECK (total_amount >= 0),
  total_hours NUMERIC(10, 2) NOT NULL CHECK (total_hours >= 0),
  hourly_rate NUMERIC(10, 2) NOT NULL CHECK (hourly_rate >= 0),
  
  period_start DATE NOT NULL,
  period_end DATE NOT NULL,
  issue_date DATE NOT NULL DEFAULT CURRENT_DATE,
  
  invoice_number TEXT,
  invoice_file_url TEXT,
  invoice_issued_at TIMESTAMPTZ,
  
  notes TEXT,
  
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  
  CONSTRAINT purchase_orders_period_check CHECK (period_end >= period_start),
  CONSTRAINT purchase_orders_invoice_check CHECK (
    (status = 'invoiced' AND invoice_number IS NOT NULL) OR 
    (status != 'invoiced')
  )
);

-- Índices
CREATE INDEX idx_purchase_orders_tenant_user 
ON purchase_orders(tenant_id, user_id);

CREATE INDEX idx_purchase_orders_status 
ON purchase_orders(status);

CREATE INDEX idx_purchase_orders_period 
ON purchase_orders(period_start, period_end);

CREATE INDEX idx_purchase_orders_issue_date 
ON purchase_orders(issue_date);

CREATE INDEX idx_purchase_orders_user_status 
ON purchase_orders(user_id, status);

-- RLS Policies
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

-- Trigger para updated_at
CREATE TRIGGER set_purchase_orders_updated_at
  BEFORE UPDATE ON purchase_orders
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

COMMENT ON TABLE purchase_orders IS 'Pedidos de compra gerados após aprovação de horas de colaboradores PJ';
COMMENT ON COLUMN purchase_orders.status IS 'pending_invoice: aguardando emissão de NF, invoiced: NF emitida, cancelled: cancelado';
```

### Migration 6: Função de Validação de Geolocalização

```sql
-- Migration: 20260129_006_create_geolocation_validation_function
-- Description: Cria função para validar se uma coordenada está dentro de uma zona permitida

CREATE OR REPLACE FUNCTION validate_geolocation(
  p_user_id UUID,
  p_tenant_id UUID,
  p_latitude NUMERIC,
  p_longitude NUMERIC
) RETURNS BOOLEAN AS $$
DECLARE
  v_zone RECORD;
  v_distance NUMERIC;
BEGIN
  -- Se não há zonas definidas para o usuário ou tenant, permite a batida
  IF NOT EXISTS (
    SELECT 1 FROM geolocation_zones 
    WHERE tenant_id = p_tenant_id 
    AND (user_id = p_user_id OR user_id IS NULL)
    AND is_active = true
  ) THEN
    RETURN true;
  END IF;
  
  -- Verifica se está dentro de alguma zona permitida
  FOR v_zone IN 
    SELECT center_latitude, center_longitude, radius_meters
    FROM geolocation_zones
    WHERE tenant_id = p_tenant_id
    AND (user_id = p_user_id OR user_id IS NULL)
    AND is_active = true
  LOOP
    -- Cálculo de distância usando fórmula de Haversine (aproximada)
    v_distance := 6371000 * acos(
      cos(radians(p_latitude)) * 
      cos(radians(v_zone.center_latitude)) * 
      cos(radians(v_zone.center_longitude) - radians(p_longitude)) + 
      sin(radians(p_latitude)) * 
      sin(radians(v_zone.center_latitude))
    );
    
    IF v_distance <= v_zone.radius_meters THEN
      RETURN true;
    END IF;
  END LOOP;
  
  RETURN false;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

COMMENT ON FUNCTION validate_geolocation IS 'Valida se uma coordenada está dentro de uma zona permitida para o usuário';
```

### Migration 7: Trigger para Cálculo Automático de Valores

```sql
-- Migration: 20260129_007_create_auto_calculate_values_trigger
-- Description: Cria trigger para calcular automaticamente valores de registros PJ

CREATE OR REPLACE FUNCTION calculate_timesheet_values()
RETURNS TRIGGER AS $$
DECLARE
  v_project_rate NUMERIC;
  v_user_rate NUMERIC;
  v_duration_hours NUMERIC;
BEGIN
  -- Apenas para registros PJ
  IF NEW.project_id IS NOT NULL THEN
    -- Buscar taxa do projeto
    SELECT hourly_rate INTO v_project_rate
    FROM timesheet_projects
    WHERE id = NEW.project_id;
    
    -- Buscar taxa do usuário
    SELECT hourly_rate INTO v_user_rate
    FROM profiles
    WHERE id = NEW.user_id;
    
    -- Definir taxa (prioridade: registro > projeto > usuário)
    IF NEW.hourly_rate IS NULL THEN
      NEW.hourly_rate := COALESCE(v_project_rate, v_user_rate, 0);
    END IF;
    
    -- Calcular duração em horas
    IF NEW.total_minutes IS NOT NULL THEN
      v_duration_hours := NEW.total_minutes / 60.0;
      
      -- Calcular valor total
      NEW.total_value := NEW.hourly_rate * v_duration_hours;
    END IF;
  END IF;
  
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_calculate_timesheet_values
  BEFORE INSERT OR UPDATE ON timesheet_entries
  FOR EACH ROW
  EXECUTE FUNCTION calculate_timesheet_values();

COMMENT ON FUNCTION calculate_timesheet_values IS 'Calcula automaticamente hourly_rate e total_value para registros PJ';
```

---

## 3. VIEWS MATERIALIZADAS (Para Performance)

### View: Resumo Mensal por Usuário

```sql
-- Migration: 20260129_008_create_monthly_summary_view
-- Description: Cria view materializada para resumo mensal de horas

CREATE MATERIALIZED VIEW IF NOT EXISTS timesheet_monthly_summary_mv AS
SELECT 
  tenant_id,
  user_id,
  EXTRACT(YEAR FROM entry_date) AS year,
  EXTRACT(MONTH FROM entry_date) AS month,
  COUNT(*) AS total_entries,
  SUM(total_minutes) / 60.0 AS total_hours,
  SUM(CASE WHEN status = 'approved' THEN total_minutes ELSE 0 END) / 60.0 AS approved_hours,
  SUM(CASE WHEN status = 'pending' THEN total_minutes ELSE 0 END) / 60.0 AS pending_hours,
  SUM(CASE WHEN status = 'rejected' THEN total_minutes ELSE 0 END) / 60.0 AS rejected_hours,
  SUM(COALESCE(total_value, 0)) AS total_value,
  SUM(CASE WHEN status = 'approved' THEN COALESCE(total_value, 0) ELSE 0 END) AS approved_value,
  MAX(entry_date) AS last_entry_date
FROM timesheet_entries
GROUP BY tenant_id, user_id, EXTRACT(YEAR FROM entry_date), EXTRACT(MONTH FROM entry_date);

-- Índices na view materializada
CREATE UNIQUE INDEX idx_monthly_summary_mv_unique 
ON timesheet_monthly_summary_mv(tenant_id, user_id, year, month);

CREATE INDEX idx_monthly_summary_mv_user 
ON timesheet_monthly_summary_mv(user_id);

CREATE INDEX idx_monthly_summary_mv_period 
ON timesheet_monthly_summary_mv(year, month);

-- Função para refresh automático
CREATE OR REPLACE FUNCTION refresh_monthly_summary()
RETURNS TRIGGER AS $$
BEGIN
  REFRESH MATERIALIZED VIEW CONCURRENTLY timesheet_monthly_summary_mv;
  RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Trigger para refresh após mudanças
CREATE TRIGGER trigger_refresh_monthly_summary
  AFTER INSERT OR UPDATE OR DELETE ON timesheet_entries
  FOR EACH STATEMENT
  EXECUTE FUNCTION refresh_monthly_summary();

COMMENT ON MATERIALIZED VIEW timesheet_monthly_summary_mv IS 'Resumo mensal de horas por usuário (atualizado automaticamente)';
```

---

## 4. FUNÇÕES AUXILIARES

### Função: Detectar Sobreposição de Registros

```sql
-- Migration: 20260129_009_create_overlap_detection_function
-- Description: Detecta sobreposição de registros de timesheet

CREATE OR REPLACE FUNCTION detect_timesheet_overlap(
  p_user_id UUID,
  p_clock_in TIMESTAMPTZ,
  p_clock_out TIMESTAMPTZ,
  p_exclude_id UUID DEFAULT NULL
) RETURNS BOOLEAN AS $$
BEGIN
  RETURN EXISTS (
    SELECT 1
    FROM timesheet_entries
    WHERE user_id = p_user_id
    AND (p_exclude_id IS NULL OR id != p_exclude_id)
    AND clock_out IS NOT NULL
    AND (
      (clock_in <= p_clock_in AND clock_out > p_clock_in) OR
      (clock_in < p_clock_out AND clock_out >= p_clock_out) OR
      (clock_in >= p_clock_in AND clock_out <= p_clock_out)
    )
  );
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

COMMENT ON FUNCTION detect_timesheet_overlap IS 'Detecta se um novo registro sobrepõe registros existentes do mesmo usuário';
```

---

Este documento continua na próxima seção com a tradução de componentes React.
