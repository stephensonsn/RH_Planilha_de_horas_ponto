# Plano Completo de Integração: Módulo de Timesheet e Batida de Ponto

**Projeto:** Humanamente - IMEP  
**Módulo:** Substituição do sistema de horas e batida de ponto  
**Autor:** Manus AI  
**Data:** 29 de Janeiro de 2026

---

## SUMÁRIO EXECUTIVO

Este documento apresenta a arquitetura completa para substituir o módulo de horas e batida de ponto do projeto Humanamente, integrando as melhores práticas dos sistemas **Kimai** (para colaboradores PJ) e **MarcaPonto** (para colaboradores CLT). A solução foi projetada para ser implementada no Lovable, utilizando React, TypeScript, Supabase e Tailwind CSS, mantendo total fidelidade visual e funcional aos sistemas de referência.

### Decisão Estratégica

**Não integrar os sistemas externos diretamente.** A abordagem recomendada é **traduzir** os conceitos, interfaces e fluxos para o ecossistema tecnológico do Humanamente, garantindo:

- **Performance superior:** Sem overhead de integração com sistemas externos
- **Manutenibilidade:** Código unificado e controlável
- **Estética consistente:** Design system único com Tailwind CSS
- **Controle total:** Personalização sem limitações de APIs externas

---

## 1. ARQUITETURA VISUAL E WIREFRAMES

### 1.1 Interface PJ (Inspirada no Kimai)

**Referência Visual:**

![Kimai Dashboard](./assets/Gfbq4ui7hgHL.webp)

#### Elementos da Interface

**Header Global:**
- Logo "Humanamente" (canto superior esquerdo)
- Timer ativo (se houver registro em andamento) com botão de parar
- Ícone de notificações
- Avatar do usuário com dropdown (Perfil, Configurações, Sair)

**Sidebar de Navegação:**
- Dashboard
- **Apontamento de Horas** (Time Tracking)
  - Meus Apontamentos
  - Horas Semanais
  - Calendário
- Projetos
- Relatórios
- Configurações

**Tela Principal: "Meus Apontamentos"**

```
┌─────────────────────────────────────────────────────────────────┐
│ Meus Apontamentos                    [Filtros] [+ Novo]         │
├─────────────────────────────────────────────────────────────────┤
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │ Timer Ativo: Projeto X - Atividade Y    [02:34:12] [STOP]  │ │
│ └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
│ ┌───────────────────────────────────────────────────────────┐   │
│ │ Data     │ Início │ Fim   │ Duração │ Projeto │ Atividade │   │
│ ├───────────────────────────────────────────────────────────┤   │
│ │ 28/01/26 │ 09:00  │ 12:30 │ 3h 30m  │ IMEP    │ Dev       │   │
│ │ 28/01/26 │ 14:00  │ 18:15 │ 4h 15m  │ IMEP    │ Reunião   │   │
│ │ 27/01/26 │ 10:00  │ 13:00 │ 3h 00m  │ Proj A  │ Análise   │   │
│ └───────────────────────────────────────────────────────────┘   │
│                                                                   │
│ Total da Semana: 42h 15m                                         │
│ [👁️ Mostrar Valor] Valor Projetado: R$ 4.225,00                 │
└─────────────────────────────────────────────────────────────────┘
```

**Paleta de Cores (Kimai-inspired):**
- Primária: `#0d47a1` (Azul escuro)
- Secundária: `#1976d2` (Azul médio)
- Sucesso: `#4caf50` (Verde)
- Alerta: `#ff9800` (Laranja)
- Erro: `#f44336` (Vermelho)
- Background: `#f5f5f5` (Cinza claro)
- Sidebar: `#263238` (Cinza escuro)

### 1.2 Interface CLT (Inspirada no MarcaPonto)

**Referência Visual:**

![MarcaPonto Interface](./assets/marcaponto_home.png)

#### Elementos da Interface

**Header Global:**
- Logo "Humanamente"
- Relógio em tempo real
- Avatar do usuário

**Tela Principal: "Bater Ponto"**

```
┌─────────────────────────────────────────────────────────────────┐
│                        HUMANAMENTE                               │
│                                                                   │
│                    ┌───────────────────┐                         │
│                    │   🕐  14:32:45    │                         │
│                    │ Quarta, 29 Jan    │                         │
│                    └───────────────────┘                         │
│                                                                   │
│              ┌─────────────────────────────┐                     │
│              │                             │                     │
│              │    [MARCAR PONTO]           │                     │
│              │                             │                     │
│              └─────────────────────────────┘                     │
│                                                                   │
│                    📍 Localização capturada                      │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Últimas Batidas                                         │    │
│  ├─────────────────────────────────────────────────────────┤    │
│  │ 29/01 - 08:00 (Entrada)                                 │    │
│  │ 29/01 - 12:00 (Saída para Almoço)                       │    │
│  │ 29/01 - 13:00 (Retorno do Almoço)                       │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                   │
│  Total Trabalhado Hoje: 5h 00m                                   │
└─────────────────────────────────────────────────────────────────┘
```

**Paleta de Cores (MarcaPonto-inspired):**
- Primária: `#8b0000` (Vermelho escuro/bordô)
- Secundária: `#c41e3a` (Vermelho)
- Neutro: `#f0f0f0` (Cinza claro)
- Texto: `#333333` (Cinza escuro)
- Background: `#ffffff` (Branco)

---

## 2. ESTRUTURA DE BANCO DE DADOS COMPLETA

### 2.1 Diagrama Entidade-Relacionamento

```
┌─────────────┐         ┌──────────────────┐         ┌─────────────┐
│   tenants   │────────<│    profiles      │>────────│   projects  │
└─────────────┘         └──────────────────┘         └─────────────┘
                                │                            │
                                │                            │
                                ▼                            ▼
                        ┌──────────────────┐         ┌─────────────┐
                        │ timesheet_entries│────────<│ activities  │
                        └──────────────────┘         └─────────────┘
                                │
                                │
                                ▼
                        ┌──────────────────┐
                        │ purchase_orders  │
                        └──────────────────┘
```

### 2.2 Tabelas e Campos Detalhados

#### Tabela: `profiles` (Extensão de `auth.users`)

```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  full_name TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  avatar_url TEXT,
  role TEXT NOT NULL CHECK (role IN ('admin', 'manager', 'user')),
  contract_type TEXT NOT NULL CHECK (contract_type IN ('CLT', 'PJ')),
  hourly_rate DECIMAL(10,2), -- Valor/hora base (pode ser sobrescrito por projeto)
  department TEXT,
  position TEXT,
  manager_id UUID REFERENCES profiles(id), -- Gestor direto
  is_active BOOLEAN DEFAULT true,
  show_financial_data BOOLEAN DEFAULT false, -- Toggle para mostrar valores
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_profiles_tenant ON profiles(tenant_id);
CREATE INDEX idx_profiles_manager ON profiles(manager_id);
CREATE INDEX idx_profiles_contract_type ON profiles(contract_type);
```

#### Tabela: `timesheet_projects`

```sql
CREATE TABLE timesheet_projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  name TEXT NOT NULL,
  description TEXT,
  client_name TEXT,
  color TEXT DEFAULT '#1976d2', -- Para visualização no calendário
  hourly_rate DECIMAL(10,2), -- Sobrescreve o hourly_rate do profile
  budget_hours DECIMAL(10,2),
  budget_amount DECIMAL(10,2),
  start_date DATE,
  end_date DATE,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_projects_tenant ON timesheet_projects(tenant_id);
CREATE INDEX idx_projects_active ON timesheet_projects(is_active);
```

#### Tabela: `timesheet_activities`

```sql
CREATE TABLE timesheet_activities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  project_id UUID REFERENCES timesheet_projects(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  description TEXT,
  is_billable BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_activities_project ON timesheet_activities(project_id);
```

#### Tabela: `timesheet_entries` (Núcleo do Sistema)

```sql
CREATE TABLE timesheet_entries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  user_id UUID NOT NULL REFERENCES profiles(id),
  project_id UUID REFERENCES timesheet_projects(id),
  activity_id UUID REFERENCES timesheet_activities(id),
  
  -- Campos de tempo
  entry_date DATE NOT NULL DEFAULT CURRENT_DATE,
  clock_in TIMESTAMPTZ NOT NULL,
  clock_out TIMESTAMPTZ,
  total_minutes INTEGER GENERATED ALWAYS AS (
    EXTRACT(EPOCH FROM (clock_out - clock_in)) / 60
  ) STORED,
  
  -- Campos específicos para CLT
  entry_type TEXT CHECK (entry_type IN ('clock_in', 'break_start', 'break_end', 'clock_out')),
  clock_in_latitude DECIMAL(10,8),
  clock_in_longitude DECIMAL(11,8),
  clock_out_latitude DECIMAL(10,8),
  clock_out_longitude DECIMAL(11,8),
  device_info JSONB, -- {ip, user_agent, device_id}
  
  -- Campos financeiros (PJ)
  hourly_rate DECIMAL(10,2), -- Cascata: entry > project > profile
  total_value DECIMAL(10,2) GENERATED ALWAYS AS (
    (total_minutes / 60.0) * COALESCE(hourly_rate, 0)
  ) STORED,
  is_billable BOOLEAN DEFAULT true,
  
  -- Campos de controle
  status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('draft', 'pending', 'approved', 'rejected', 'invoiced')),
  rejection_reason TEXT,
  approved_by UUID REFERENCES profiles(id),
  approved_at TIMESTAMPTZ,
  
  -- Metadados
  description TEXT,
  tags TEXT[],
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  CONSTRAINT check_clock_out_after_in CHECK (clock_out IS NULL OR clock_out > clock_in),
  CONSTRAINT check_max_duration CHECK (total_minutes IS NULL OR total_minutes <= 720) -- 12 horas
);

CREATE INDEX idx_entries_user ON timesheet_entries(user_id);
CREATE INDEX idx_entries_project ON timesheet_entries(project_id);
CREATE INDEX idx_entries_date ON timesheet_entries(entry_date);
CREATE INDEX idx_entries_status ON timesheet_entries(status);
CREATE INDEX idx_entries_tenant_date ON timesheet_entries(tenant_id, entry_date);
```

#### Tabela: `geolocation_zones`

```sql
CREATE TABLE geolocation_zones (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  user_id UUID REFERENCES profiles(id), -- NULL = zona global para o tenant
  name TEXT NOT NULL,
  zone_polygon GEOGRAPHY(POLYGON, 4326) NOT NULL, -- PostGIS
  radius_meters INTEGER DEFAULT 100,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_zones_user ON geolocation_zones(user_id);
CREATE INDEX idx_zones_polygon ON geolocation_zones USING GIST(zone_polygon);
```

#### Tabela: `purchase_orders`

```sql
CREATE TABLE purchase_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  user_id UUID NOT NULL REFERENCES profiles(id),
  period_start DATE NOT NULL,
  period_end DATE NOT NULL,
  total_hours DECIMAL(10,2) NOT NULL,
  total_amount DECIMAL(10,2) NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending_invoice' CHECK (status IN ('pending_invoice', 'invoice_requested', 'invoiced', 'paid')),
  invoice_file_url TEXT,
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_po_user ON purchase_orders(user_id);
CREATE INDEX idx_po_status ON purchase_orders(status);
```

### 2.3 Funções e Triggers

#### Função: `validate_geolocation`

```sql
CREATE OR REPLACE FUNCTION validate_geolocation(
  p_user_id UUID,
  p_tenant_id UUID,
  p_latitude DECIMAL,
  p_longitude DECIMAL
) RETURNS BOOLEAN AS $$
DECLARE
  v_point GEOGRAPHY;
  v_zone_count INTEGER;
BEGIN
  v_point := ST_SetSRID(ST_MakePoint(p_longitude, p_latitude), 4326)::GEOGRAPHY;
  
  SELECT COUNT(*) INTO v_zone_count
  FROM geolocation_zones
  WHERE tenant_id = p_tenant_id
    AND (user_id = p_user_id OR user_id IS NULL)
    AND is_active = true
    AND ST_DWithin(zone_polygon, v_point, radius_meters);
  
  RETURN v_zone_count > 0;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

#### Trigger: `check_overlapping_entries`

```sql
CREATE OR REPLACE FUNCTION fn_check_overlapping_entries()
RETURNS TRIGGER AS $$
DECLARE
  v_overlap_count INTEGER;
BEGIN
  IF NEW.clock_out IS NOT NULL THEN
    SELECT COUNT(*) INTO v_overlap_count
    FROM timesheet_entries
    WHERE user_id = NEW.user_id
      AND id != COALESCE(NEW.id, '00000000-0000-0000-0000-000000000000'::UUID)
      AND clock_out IS NOT NULL
      AND (
        (NEW.clock_in, NEW.clock_out) OVERLAPS (clock_in, clock_out)
      );
    
    IF v_overlap_count > 0 THEN
      RAISE EXCEPTION 'Time entry overlaps with an existing one.';
    END IF;
  END IF;
  
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_check_overlapping
  BEFORE INSERT OR UPDATE ON timesheet_entries
  FOR EACH ROW
  EXECUTE FUNCTION fn_check_overlapping_entries();
```

---

## 3. COMPONENTES REACT TRADUZIDOS

### 3.1 Componente: `TimesheetRouter.tsx`

```tsx
// src/components/timesheet/TimesheetRouter.tsx

import React from 'react';
import { useProfile } from '@/hooks/useProfile';
import TimesheetPJ from './pj/TimesheetPJ';
import TimesheetCLT from './clt/TimesheetCLT';
import { Loader2 } from 'lucide-react';

const TimesheetRouter: React.FC = () => {
  const { profile, isLoading } = useProfile();

  if (isLoading) {
    return (
      <div className="flex items-center justify-center h-screen">
        <Loader2 className="w-8 h-8 animate-spin text-primary" />
      </div>
    );
  }

  if (!profile) {
    return <div>Erro ao carregar perfil</div>;
  }

  return profile.contract_type === 'PJ' ? <TimesheetPJ /> : <TimesheetCLT />;
};

export default TimesheetRouter;
```

### 3.2 Componente: `ClockInCard.tsx` (CLT)

```tsx
// src/components/timesheet/clt/ClockInCard.tsx

import React, { useState, useEffect } from 'react';
import { useGeolocated } from 'react-geolocated';
import { Clock, MapPin, AlertCircle } from 'lucide-react';
import { useSupabaseClient, useUser } from '@supabase/auth-helpers-react';
import { toast } from 'sonner';

const ClockInCard: React.FC = () => {
  const [currentTime, setCurrentTime] = useState(new Date());
  const [isSubmitting, setIsSubmitting] = useState(false);
  const supabase = useSupabaseClient();
  const user = useUser();
  
  const { coords, isGeolocationAvailable, isGeolocationEnabled } = useGeolocated({
    positionOptions: { enableHighAccuracy: true },
    userDecisionTimeout: 5000,
  });

  useEffect(() => {
    const timer = setInterval(() => setCurrentTime(new Date()), 1000);
    return () => clearInterval(timer);
  }, []);

  const handleClockIn = async () => {
    if (!isGeolocationEnabled || !coords) {
      toast.error('Por favor, habilite a geolocalização para bater o ponto.');
      return;
    }

    setIsSubmitting(true);
    try {
      // 1. Validar geolocalização
      const { data: isValid, error: validationError } = await supabase.rpc('validate_geolocation', {
        p_user_id: user?.id,
        p_tenant_id: user?.user_metadata?.tenant_id,
        p_latitude: coords.latitude,
        p_longitude: coords.longitude,
      });

      if (validationError || !isValid) {
        toast.error('Você está fora da área permitida para bater o ponto.');
        return;
      }

      // 2. Inserir registro
      const { error: insertError } = await supabase.from('timesheet_entries').insert({
        user_id: user?.id,
        tenant_id: user?.user_metadata?.tenant_id,
        entry_type: 'clock_in', // Lógica para determinar o tipo
        clock_in: new Date().toISOString(),
        clock_in_latitude: coords.latitude,
        clock_in_longitude: coords.longitude,
        device_info: {
          user_agent: navigator.userAgent,
          // ip será capturado no backend via Edge Function
        },
        status: 'pending',
      });

      if (insertError) throw insertError;

      toast.success('Ponto registrado com sucesso!');
    } catch (error) {
      console.error('Erro ao registrar ponto:', error);
      toast.error('Erro ao registrar ponto. Tente novamente.');
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <div className="bg-white rounded-2xl shadow-xl p-8 max-w-md mx-auto text-center">
      <Clock className="w-16 h-16 text-primary mx-auto mb-4" />
      <h1 className="text-5xl font-bold text-gray-800 mb-2">
        {currentTime.toLocaleTimeString('pt-BR')}
      </h1>
      <p className="text-gray-500 mb-8">
        {currentTime.toLocaleDateString('pt-BR', { dateStyle: 'full' })}
      </p>
      
      <button
        onClick={handleClockIn}
        disabled={isSubmitting || !isGeolocationEnabled}
        className="w-full bg-primary hover:bg-primary-dark text-white font-semibold py-4 px-6 rounded-lg transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
      >
        {isSubmitting ? 'Registrando...' : 'MARCAR PONTO'}
      </button>

      <div className="flex items-center justify-center mt-6 text-sm">
        {isGeolocationEnabled ? (
          <>
            <MapPin className="w-4 h-4 text-green-500 mr-2" />
            <span className="text-green-600">Localização capturada</span>
          </>
        ) : (
          <>
            <AlertCircle className="w-4 h-4 text-red-500 mr-2" />
            <span className="text-red-600">Habilite a localização</span>
          </>
        )}
      </div>
    </div>
  );
};

export default ClockInCard;
```

### 3.3 Componente: `TimesheetTable.tsx` (PJ)

```tsx
// src/components/timesheet/pj/TimesheetTable.tsx

import React, { useMemo } from 'react';
import { useTable, useSortBy, usePagination } from 'react-table';
import { format } from 'date-fns';
import { ptBR } from 'date-fns/locale';
import { Filter, Plus, Play, Square } from 'lucide-react';

interface TimesheetEntry {
  id: string;
  entry_date: string;
  clock_in: string;
  clock_out: string | null;
  total_minutes: number | null;
  project_name: string;
  activity_name: string;
  total_value: number;
  status: string;
}

interface Props {
  data: TimesheetEntry[];
  isLoading: boolean;
  onEdit: (entry: TimesheetEntry) => void;
  onDelete: (entry: TimesheetEntry) => void;
  onStartTimer: () => void;
  activeTimer: TimesheetEntry | null;
}

const TimesheetTable: React.FC<Props> = ({ 
  data, 
  isLoading, 
  onEdit, 
  onDelete, 
  onStartTimer,
  activeTimer 
}) => {
  const columns = useMemo(() => [
    {
      Header: 'Data',
      accessor: 'entry_date',
      Cell: ({ value }: { value: string }) => format(new Date(value), 'dd/MM/yyyy', { locale: ptBR }),
    },
    {
      Header: 'Início',
      accessor: 'clock_in',
      Cell: ({ value }: { value: string }) => format(new Date(value), 'HH:mm'),
    },
    {
      Header: 'Fim',
      accessor: 'clock_out',
      Cell: ({ value }: { value: string | null }) => 
        value ? format(new Date(value), 'HH:mm') : <span className="text-blue-600">Em andamento</span>,
    },
    {
      Header: 'Duração',
      accessor: 'total_minutes',
      Cell: ({ value }: { value: number | null }) => 
        value ? `${Math.floor(value / 60)}h ${value % 60}m` : '-',
    },
    {
      Header: 'Projeto',
      accessor: 'project_name',
    },
    {
      Header: 'Atividade',
      accessor: 'activity_name',
    },
    {
      Header: 'Valor',
      accessor: 'total_value',
      Cell: ({ value }: { value: number }) => `R$ ${value.toFixed(2)}`,
    },
    {
      Header: 'Status',
      accessor: 'status',
      Cell: ({ value }: { value: string }) => {
        const statusColors = {
          draft: 'bg-gray-200 text-gray-700',
          pending: 'bg-yellow-200 text-yellow-800',
          approved: 'bg-green-200 text-green-800',
          rejected: 'bg-red-200 text-red-800',
        };
        return (
          <span className={`px-2 py-1 rounded-full text-xs font-semibold ${statusColors[value as keyof typeof statusColors]}`}>
            {value}
          </span>
        );
      },
    },
  ], []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    page,
    prepareRow,
    canPreviousPage,
    canNextPage,
    pageOptions,
    pageCount,
    gotoPage,
    nextPage,
    previousPage,
    state: { pageIndex },
  } = useTable(
    {
      columns,
      data,
      initialState: { pageIndex: 0, pageSize: 15 },
    },
    useSortBy,
    usePagination
  );

  return (
    <div className="bg-white rounded-lg shadow-md p-6">
      {/* Header */}
      <div className="flex justify-between items-center mb-6">
        <h2 className="text-2xl font-semibold text-gray-800">Meus Apontamentos</h2>
        <div className="flex items-center space-x-3">
          <button className="flex items-center px-4 py-2 border border-gray-300 rounded-lg hover:bg-gray-50 transition-colors">
            <Filter className="w-4 h-4 mr-2" />
            Filtros
          </button>
          <button 
            onClick={onStartTimer}
            className="flex items-center px-4 py-2 bg-primary text-white rounded-lg hover:bg-primary-dark transition-colors"
          >
            <Plus className="w-4 h-4 mr-2" />
            Novo Apontamento
          </button>
        </div>
      </div>

      {/* Active Timer */}
      {activeTimer && (
        <div className="bg-blue-50 border-l-4 border-blue-500 p-4 mb-6 rounded-r-lg">
          <div className="flex items-center justify-between">
            <div className="flex items-center">
              <Play className="w-5 h-5 text-blue-600 mr-3" />
              <div>
                <p className="font-semibold text-blue-900">Timer Ativo</p>
                <p className="text-sm text-blue-700">
                  {activeTimer.project_name} - {activeTimer.activity_name}
                </p>
              </div>
            </div>
            <div className="flex items-center space-x-4">
              <span className="text-2xl font-mono font-bold text-blue-900">
                {/* Timer component aqui */}
                02:34:12
              </span>
              <button className="flex items-center px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-red-600 transition-colors">
                <Square className="w-4 h-4 mr-2" />
                Parar
              </button>
            </div>
          </div>
        </div>
      )}

      {/* Table */}
      <div className="overflow-x-auto">
        <table {...getTableProps()} className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            {headerGroups.map(headerGroup => (
              <tr {...headerGroup.getHeaderGroupProps()}>
                {headerGroup.headers.map(column => (
                  <th
                    {...column.getHeaderProps(column.getSortByToggleProps())}
                    className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider cursor-pointer hover:bg-gray-100"
                  >
                    {column.render('Header')}
                  </th>
                ))}
              </tr>
            ))}
          </thead>
          <tbody {...getTableBodyProps()} className="bg-white divide-y divide-gray-200">
            {page.map(row => {
              prepareRow(row);
              return (
                <tr {...row.getRowProps()} className="hover:bg-gray-50">
                  {row.cells.map(cell => (
                    <td {...cell.getCellProps()} className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                      {cell.render('Cell')}
                    </td>
                  ))}
                </tr>
              );
            })}
          </tbody>
        </table>
      </div>

      {/* Pagination */}
      <div className="flex items-center justify-between mt-6">
        <div className="text-sm text-gray-700">
          Página <span className="font-semibold">{pageIndex + 1}</span> de{' '}
          <span className="font-semibold">{pageOptions.length}</span>
        </div>
        <div className="flex space-x-2">
          <button
            onClick={() => previousPage()}
            disabled={!canPreviousPage}
            className="px-4 py-2 border border-gray-300 rounded-lg hover:bg-gray-50 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            Anterior
          </button>
          <button
            onClick={() => nextPage()}
            disabled={!canNextPage}
            className="px-4 py-2 border border-gray-300 rounded-lg hover:bg-gray-50 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            Próxima
          </button>
        </div>
      </div>
    </div>
  );
};

export default TimesheetTable;
```

---

## 4. FLUXOS DE APROVAÇÃO E INTEGRAÇÕES

### 4.1 Edge Functions do Supabase

#### Edge Function: `on-pending-entry`

```typescript
// supabase/functions/on-pending-entry/index.ts

import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL') ?? '',
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY') ?? ''
  )

  const { record } = await req.json()

  // Buscar o gestor do usuário
  const { data: profile } = await supabase
    .from('profiles')
    .select('manager_id, full_name')
    .eq('id', record.user_id)
    .single()

  if (!profile?.manager_id) {
    return new Response('No manager found', { status: 200 })
  }

  const { data: manager } = await supabase
    .from('profiles')
    .select('email, full_name')
    .eq('id', profile.manager_id)
    .single()

  // Enviar e-mail via Resend
  const RESEND_API_KEY = Deno.env.get('RESEND_API_KEY')
  
  await fetch('https://api.resend.com/emails', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${RESEND_API_KEY}`,
    },
    body: JSON.stringify({
      from: 'Humanamente <noreply@humanamente.com>',
      to: manager.email,
      subject: `Aprovação de Horas - ${profile.full_name}`,
      html: `
        <h2>Olá, ${manager.full_name}</h2>
        <p>${profile.full_name} submeteu horas para aprovação.</p>
        <a href="${Deno.env.get('APP_URL')}/aprovacoes">Clique aqui para revisar</a>
      `,
    }),
  })

  return new Response('OK', { status: 200 })
})
```

#### Edge Function: `on-approved-pj-entry`

```typescript
// supabase/functions/on-approved-pj-entry/index.ts

import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL') ?? '',
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY') ?? ''
  )

  const { record } = await req.json()

  // Verificar se é PJ
  const { data: profile } = await supabase
    .from('profiles')
    .select('contract_type')
    .eq('id', record.user_id)
    .single()

  if (profile?.contract_type !== 'PJ') {
    return new Response('Not a PJ user', { status: 200 })
  }

  // Agregar todas as horas aprovadas do mês
  const startOfMonth = new Date(record.entry_date)
  startOfMonth.setDate(1)
  
  const endOfMonth = new Date(startOfMonth)
  endOfMonth.setMonth(endOfMonth.getMonth() + 1)
  endOfMonth.setDate(0)

  const { data: entries } = await supabase
    .from('timesheet_entries')
    .select('total_minutes, total_value')
    .eq('user_id', record.user_id)
    .eq('status', 'approved')
    .gte('entry_date', startOfMonth.toISOString())
    .lte('entry_date', endOfMonth.toISOString())

  const totalHours = entries?.reduce((sum, e) => sum + (e.total_minutes || 0), 0) / 60 || 0
  const totalAmount = entries?.reduce((sum, e) => sum + (e.total_value || 0), 0) || 0

  // Criar ou atualizar Pedido de Compra
  const { error } = await supabase
    .from('purchase_orders')
    .upsert({
      tenant_id: record.tenant_id,
      user_id: record.user_id,
      period_start: startOfMonth.toISOString().split('T')[0],
      period_end: endOfMonth.toISOString().split('T')[0],
      total_hours: totalHours,
      total_amount: totalAmount,
      status: 'pending_invoice',
    }, {
      onConflict: 'user_id,period_start',
    })

  if (error) {
    console.error('Error creating purchase order:', error)
  }

  return new Response('OK', { status: 200 })
})
```

### 4.2 Configuração de Database Webhooks

No painel do Supabase, configure os seguintes webhooks:

1.  **Webhook: `on_pending_entry`**
    - **Tabela:** `timesheet_entries`
    - **Evento:** `INSERT` ou `UPDATE`
    - **Condição:** `NEW.status = 'pending' AND OLD.status != 'pending'`
    - **Função:** `on-pending-entry`

2.  **Webhook: `on_approved_pj_entry`**
    - **Tabela:** `timesheet_entries`
    - **Evento:** `UPDATE`
    - **Condição:** `NEW.status = 'approved' AND OLD.status != 'approved'`
    - **Função:** `on-approved-pj-entry`

---

## 5. CHECKLIST DE IMPLEMENTAÇÃO

### Fase 1: Estrutura de Backend (Supabase)

- [ ] Criar todas as tabelas no Supabase (migrations)
- [ ] Configurar Row Level Security (RLS) para cada tabela
- [ ] Implementar funções PostgreSQL (`validate_geolocation`, etc.)
- [ ] Criar triggers (`check_overlapping_entries`)
- [ ] Configurar extensão PostGIS para geolocalização
- [ ] Testar todas as queries e validações

### Fase 2: Componentes de Interface (Frontend)

- [ ] Criar `TimesheetRouter` para diferenciar PJ/CLT
- [ ] Implementar `ClockInCard` (CLT) com geolocalização
- [ ] Implementar `TimesheetTable` (PJ) com timer e filtros
- [ ] Criar formulários de edição de apontamentos
- [ ] Implementar tela de aprovação para gestores
- [ ] Criar dashboard com widgets de resumo
- [ ] Adicionar toggle de visibilidade de valores financeiros

### Fase 3: Integrações e Automações

- [ ] Criar Edge Functions no Supabase
- [ ] Configurar Resend ou SendGrid para e-mails
- [ ] Implementar webhooks para notificações
- [ ] Criar fluxo de geração de Pedidos de Compra
- [ ] Implementar upload de Notas Fiscais
- [ ] Testar fluxo completo de aprovação e faturamento

### Fase 4: Testes e Refinamentos

- [ ] Testes de usabilidade com colaboradores PJ
- [ ] Testes de usabilidade com colaboradores CLT
- [ ] Testes de geolocalização em diferentes dispositivos
- [ ] Validação de cálculos financeiros
- [ ] Testes de performance com grande volume de dados
- [ ] Ajustes de UI/UX baseados em feedback

---

## 6. CONSIDERAÇÕES FINAIS

### Pontos de Atenção

1.  **Privacidade de Geolocalização:** Garantir que os colaboradores sejam informados sobre a coleta de dados de localização e obter consentimento explícito (LGPD).

2.  **Precisão de Cálculos:** Implementar testes automatizados para validar os cálculos de duração, valores e agregações.

3.  **Escalabilidade:** A estrutura de índices no banco de dados foi projetada para suportar crescimento. Monitorar performance conforme o volume de dados aumenta.

4.  **Backup e Auditoria:** Todos os registros de ponto e aprovações devem ser imutáveis após um período de graça. Implementar logs de auditoria para rastreabilidade.

### Próximos Passos Recomendados

1.  **Prototipagem:** Criar um protótipo no Figma com base nos wireframes para validação visual antes da implementação.

2.  **MVP:** Começar com a Fase 1 e 2, implementando apenas o fluxo básico de apontamento e aprovação, sem integrações complexas.

3.  **Iteração:** Coletar feedback dos usuários e iterar sobre a interface e os fluxos antes de adicionar funcionalidades avançadas.

---

**Documento elaborado por Manus AI em 29 de Janeiro de 2026.**
