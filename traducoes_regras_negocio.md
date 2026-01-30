# Tradução de Componentes e Regras de Negócio

**Autor:** Manus AI  
**Data:** 29 de Janeiro de 2026

---

## 1. TRADUÇÃO DE COMPONENTES: PJ (Kimai)

### Componente: `TimesheetTable` (Tabela de Registros)

- **Original (Kimai - Twig/JS):** `templates/timesheet/index.html.twig` + JS
- **Tradução (Humanamente - React):** `src/components/timesheet/pj/TimesheetTable.tsx`

```tsx
// src/components/timesheet/pj/TimesheetTable.tsx

import React, { useState, useMemo } from 'react';
import { useTable, useSortBy, usePagination, useRowSelect } from 'react-table';
import { format } from 'date-fns';
import { FiFilter, FiPlus, FiDownload } from 'react-icons/fi';
import { TimesheetEntry } from '@/types'; // Tipagem vinda do Supabase

interface Props {
  data: TimesheetEntry[];
  isLoading: boolean;
  onEdit: (entry: TimesheetEntry) => void;
  onDelete: (entry: TimesheetEntry) => void;
  onApprove: (selected: TimesheetEntry[]) => void;
}

const TimesheetTable: React.FC<Props> = ({ data, isLoading, onEdit, onDelete, onApprove }) => {
  const columns = useMemo(() => [
    { Header: 'Data', accessor: 'entry_date', Cell: ({ value }) => format(new Date(value), 'dd/MM/yyyy') },
    { Header: 'Início', accessor: 'clock_in', Cell: ({ value }) => format(new Date(value), 'HH:mm') },
    { Header: 'Fim', accessor: 'clock_out', Cell: ({ value }) => value ? format(new Date(value), 'HH:mm') : 'Em andamento' },
    { Header: 'Duração', accessor: 'total_minutes', Cell: ({ value }) => `${Math.floor(value / 60)}h ${value % 60}m` },
    { Header: 'Projeto', accessor: 'projects.name' },
    { Header: 'Atividade', accessor: 'activities.name' },
    { Header: 'Valor', accessor: 'total_value', Cell: ({ value }) => `R$ ${value.toFixed(2)}` },
    { Header: 'Status', accessor: 'status' },
    // ... outras colunas e ações
  ], []);

  const { getTableProps, getTableBodyProps, headerGroups, page, prepareRow } = useTable(
    { columns, data, initialState: { pageIndex: 0, pageSize: 15 } },
    useSortBy,
    usePagination,
    useRowSelect
  );

  return (
    <div className="bg-white p-6 rounded-lg shadow-md">
      <div className="flex justify-between items-center mb-4">
        <h2 className="text-xl font-semibold">Meus Apontamentos</h2>
        <div className="flex items-center space-x-2">
          <button className="btn-secondary"><FiFilter className="mr-2" /> Filtros</button>
          <button className="btn-primary"><FiPlus className="mr-2" /> Novo Apontamento</button>
        </div>
      </div>
      {/* ... implementação da tabela com react-table ... */}
    </div>
  );
};

export default TimesheetTable;
```

### Regras de Negócio Associadas

1.  **Cálculo de Duração:** `total_minutes` é calculado no backend (via trigger ou na inserção) subtraindo `clock_in` de `clock_out`.
2.  **Cálculo de Valor:** `total_value` é `(total_minutes / 60) * hourly_rate`. A `hourly_rate` é definida em cascata: `timesheet_entries.hourly_rate` > `timesheet_projects.hourly_rate` > `profiles.hourly_rate`.
3.  **Status:** O status (`pending`, `approved`, `rejected`) controla a visibilidade das ações (ex: só gestor aprova).
4.  **Timer Ativo:** Se `clock_out` for `NULL`, o registro está ativo. O header global deve mostrar um timer rodando.

---

## 2. TRADUÇÃO DE COMPONENTES: CLT (MarcaPonto)

### Componente: `ClockInCard` (Card de Batida de Ponto)

- **Original (MarcaPonto - React):** `src/Components/MarcarPonto/index.tsx`
- **Tradução (Humanamente - React):** `src/components/timesheet/clt/ClockInCard.tsx`

```tsx
// src/components/timesheet/clt/ClockInCard.tsx

import React, { useState, useEffect } from 'react';
import { useGeolocated } from 'react-geolocated';
import { FiClock, FiMapPin } from 'react-icons/fi';
import { useSupabaseClient } from '@supabase/auth-helpers-react';

interface Props {
  userId: string;
  tenantId: string;
}

const ClockInCard: React.FC<Props> = ({ userId, tenantId }) => {
  const [time, setTime] = useState(new Date());
  const [isSubmitting, setIsSubmitting] = useState(false);
  const supabase = useSupabaseClient();
  const { coords, isGeolocationAvailable, isGeolocationEnabled } = useGeolocated({
    positionOptions: { enableHighAccuracy: true },
    userDecisionTimeout: 5000,
  });

  useEffect(() => {
    const timer = setInterval(() => setTime(new Date()), 1000);
    return () => clearInterval(timer);
  }, []);

  const handleClockIn = async () => {
    if (!isGeolocationEnabled || !coords) {
      alert('Por favor, habilite a geolocalização para bater o ponto.');
      return;
    }

    setIsSubmitting(true);
    try {
      // 1. Validar se está dentro da zona permitida
      const { data: isValid, error: validationError } = await supabase.rpc('validate_geolocation', {
        p_user_id: userId,
        p_tenant_id: tenantId,
        p_latitude: coords.latitude,
        p_longitude: coords.longitude,
      });

      if (validationError || !isValid) {
        // ... tratar erro ou batida fora da zona
        return;
      }

      // 2. Inserir o registro de ponto
      const { error: insertError } = await supabase.from('timesheet_entries').insert({
        user_id: userId,
        tenant_id: tenantId,
        entry_type: 'clock_in', // Lógica para determinar o tipo de batida
        clock_in: new Date().toISOString(),
        clock_in_latitude: coords.latitude,
        clock_in_longitude: coords.longitude,
        status: 'pending',
      });

      // ... tratar sucesso
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg text-center max-w-sm mx-auto">
      <FiClock className="text-5xl text-primary mx-auto mb-4" />
      <h1 className="text-4xl font-bold">{time.toLocaleTimeString('pt-BR')}</h1>
      <p className="text-gray-500 mb-6">{time.toLocaleDateString('pt-BR', { dateStyle: 'full' })}</p>
      <button 
        onClick={handleClockIn}
        disabled={isSubmitting || !isGeolocationEnabled}
        className="btn-primary w-full text-lg py-4"
      >
        {isSubmitting ? 'Registrando...' : 'Marcar Ponto'}
      </button>
      <div className="flex items-center justify-center mt-4 text-gray-600">
        <FiMapPin className="mr-2" />
        <span>{isGeolocationEnabled ? 'Localização capturada' : 'Habilite a localização'}</span>
      </div>
    </div>
  );
};

export default ClockInCard;
```

### Regras de Negócio Associadas

1.  **Geolocalização Obrigatória:** A batida de ponto só é permitida se o usuário conceder acesso à localização.
2.  **Validação de Zona:** Antes de registrar, o sistema chama a função `validate_geolocation` no banco de dados para verificar se as coordenadas (`coords.latitude`, `coords.longitude`) estão dentro de alguma `geolocation_zones` ativa para aquele usuário/tenant.
3.  **Sequência de Batidas:** O sistema precisa determinar o `entry_type` correto. Uma lógica simples seria:
    - Se não há batidas no dia, a primeira é `clock_in`.
    - Se a última foi `clock_in`, a próxima é `break_start` ou `clock_out`.
    - Se a última foi `break_start`, a próxima é `break_end`.
    - Se a última foi `break_end`, a próxima é `clock_out`.
4.  **Auditoria:** As informações do dispositivo (IP, User-Agent) devem ser capturadas e salvas no campo `device_info` (JSONB) para fins de auditoria.

---

## 3. REGRAS DE NEGÓCIO: FLUXO DE APROVAÇÃO

1.  **Submissão:**
    - **CLT:** Cada batida individual gera um registro com status `pending`.
    - **PJ:** O colaborador trabalha nos seus registros e, ao final de um período (semanal, quinzenal, mensal), submete um conjunto de registros para aprovação. Isso pode ser uma ação que muda o status de vários registros de `draft` para `pending`.

2.  **Notificação:**
    - Uma Edge Function do Supabase é acionada por `INSERT` na tabela `timesheet_entries` com status `pending`.
    - A função identifica o gestor do usuário e envia um e-mail (via Resend/SendGrid) e uma notificação no sistema.

3.  **Aprovação/Rejeição (Gestor):**
    - O gestor visualiza os registros pendentes na tela de "Aprovações".
    - Ao clicar em "Aprovar", o status do registro muda para `approved`.
    - Ao clicar em "Rejeitar", o status muda para `rejected` e o campo `rejection_reason` é preenchido obrigatoriamente.

4.  **Pós-Aprovação (PJ):**
    - Uma outra Edge Function é acionada por `UPDATE` na tabela `timesheet_entries` onde o status muda para `approved` e `contract_type` é `PJ`.
    - A função agrega todos os registros aprovados para um usuário em um determinado período (ex: mês).
    - Ela então cria uma nova entrada na tabela `purchase_orders` com o `total_amount`, `total_hours`, e `status = 'pending_invoice'`.

5.  **Solicitação de NF:**
    - Na tela de Pedidos de Compra, o gestor ou financeiro clica em "Solicitar NF".
    - Isso aciona uma terceira Edge Function que envia um e-mail para o endereço de e-mail do financeiro (configurado em `tenants.financial_email`) com os detalhes do Pedido de Compra e um link para o colaborador fazer o upload da NF.

6.  **Finalização:**
    - Após a emissão e pagamento, o financeiro atualiza o status do `purchase_order` para `invoiced` e anexa a URL da nota fiscal no campo `invoice_file_url`.

---

Este documento continua na próxima seção com a documentação de validações e túneis de fluxo.
