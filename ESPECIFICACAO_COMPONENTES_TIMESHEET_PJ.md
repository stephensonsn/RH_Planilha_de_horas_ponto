# Especificação de Componentes React/TypeScript para Timesheet PJ (Kimai)

**Projeto:** Humanamente - Módulo de Apontamento de Horas PJ  
**Stack:** React + TypeScript + Tailwind CSS + Supabase  
**Referência Visual:** Kimai (https://demo-empty.kimai.org)  
**Data:** 01/02/2026

---

## 1. ANÁLISE COMPARATIVA: IMEP (Atual) vs Kimai (Objetivo)

### IMEP Atual (CLT - Batida de Ponto)
- Interface simplificada com botão grande "Registrar Entrada"
- Relógio em tempo real
- Geolocalização automática
- Calendário mensal com resumo de horas
- **Limitação:** Não permite apontamento por projeto/atividade

### Kimai (PJ - Apontamento de Horas)
- Timer ativo no header global
- Tabela completa com filtros avançados
- Modal de criação/edição com múltiplos campos
- Calendário interativo (mês/semana/dia)
- Associação a projetos e atividades
- Cálculo automático de valores

---

## 2. BIBLIOTECAS E COMPONENTES NECESSÁRIOS

### 2.1. Gerenciamento de Estado
```typescript
// Supabase Client (já existe no projeto)
import { createClient } from '@supabase/supabase-js'

// React Query para cache e sincronização
npm install @tanstack/react-query
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'

// Zustand para estado global (timer ativo)
npm install zustand
import { create } from 'zustand'
```

### 2.2. Componentes UI (shadcn/ui - já usado no Humanamente)
```bash
# Componentes necessários
npx shadcn-ui@latest add table
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add select
npx shadcn-ui@latest add calendar
npx shadcn-ui@latest add popover
npx shadcn-ui@latest add tabs
npx shadcn-ui@latest add badge
npx shadcn-ui@latest add dropdown-menu
npx shadcn-ui@latest add command
npx shadcn-ui@latest add input
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
npx shadcn-ui@latest add separator
```

### 2.3. Calendário Avançado
```bash
# React Big Calendar (visualização mensal/semanal/diária)
npm install react-big-calendar
npm install date-fns

# Alternativa: FullCalendar (mais completo, mas maior)
# npm install @fullcalendar/react @fullcalendar/daygrid @fullcalendar/timegrid @fullcalendar/interaction
```

### 2.4. Manipulação de Datas e Horários
```bash
# date-fns (já usado no Humanamente)
npm install date-fns

# Para formatação de duração
npm install duration-fns
```

### 2.5. Ícones
```bash
# Lucide React (já usado no Humanamente)
npm install lucide-react
```

### 2.6. Formulários e Validação
```bash
# React Hook Form (já usado no Humanamente)
npm install react-hook-form

# Zod para validação de schema
npm install zod
npm install @hookform/resolvers
```

---

## 3. ESTRUTURA DE COMPONENTES

### 3.1. Componente Principal: `TimesheetPJDashboard.tsx`
**Localização:** `/src/components/timesheet/pj/TimesheetPJDashboard.tsx`

**Responsabilidades:**
- Layout principal com header, sidebar e conteúdo
- Gerenciamento de abas (Meus Horários, Calendário, Relatórios)
- Integração com timer global

**Componentes Filhos:**
- `TimerWidget` (header global)
- `TimesheetTable` (tabela principal)
- `TimesheetCalendar` (calendário interativo)
- `TimesheetFilters` (filtros avançados)

**Wireframe:**
```
┌─────────────────────────────────────────────────────────────────┐
│ HEADER: [▶️ Timer: 0:00] [🔄 Reiniciar] [👤 Fabrizio Castro]    │
├─────────────────────────────────────────────────────────────────┤
│ SIDEBAR │ CONTEÚDO PRINCIPAL                                    │
│         │ ┌───────────────────────────────────────────────────┐ │
│ 📊 Dash │ │ 📋 Meus Horários                                  │ │
│ ⏱️ Meus │ │ ┌─────────────────────────────────────────────┐   │ │
│ 📅 Cal  │ │ │ [🔍 Buscar] [📅 Período] [📁 Projeto] [➕]  │   │ │
│ 📈 Rel  │ │ └─────────────────────────────────────────────┘   │ │
│         │ │                                                   │ │
│         │ │ [Tabela de Registros]                            │ │
│         │ └───────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

**Código Base:**
```typescript
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs'
import { TimerWidget } from './TimerWidget'
import { TimesheetTable } from './TimesheetTable'
import { TimesheetCalendar } from './TimesheetCalendar'

export function TimesheetPJDashboard() {
  return (
    <div className="flex h-screen flex-col">
      {/* Header com Timer */}
      <header className="border-b bg-background px-6 py-3">
        <div className="flex items-center justify-between">
          <h1 className="text-2xl font-bold">Planilha de Horas</h1>
          <TimerWidget />
        </div>
      </header>

      {/* Conteúdo Principal */}
      <div className="flex flex-1 overflow-hidden">
        {/* Sidebar (opcional, pode usar menu superior) */}
        <aside className="w-64 border-r bg-muted/40 p-4">
          {/* Navegação */}
        </aside>

        {/* Área de Conteúdo */}
        <main className="flex-1 overflow-y-auto p-6">
          <Tabs defaultValue="timesheet" className="space-y-4">
            <TabsList>
              <TabsTrigger value="timesheet">Meus Horários</TabsTrigger>
              <TabsTrigger value="calendar">Calendário</TabsTrigger>
              <TabsTrigger value="reports">Relatórios</TabsTrigger>
            </TabsList>

            <TabsContent value="timesheet">
              <TimesheetTable />
            </TabsContent>

            <TabsContent value="calendar">
              <TimesheetCalendar />
            </TabsContent>

            <TabsContent value="reports">
              {/* Relatórios */}
            </TabsContent>
          </Tabs>
        </main>
      </div>
    </div>
  )
}
```

---

### 3.2. Componente: `TimerWidget.tsx`
**Localização:** `/src/components/timesheet/pj/TimerWidget.tsx`

**Responsabilidades:**
- Exibir timer ativo no header
- Botões: Play/Pause, Stop, Reiniciar
- Seleção rápida de projeto/atividade
- Salvar automaticamente ao parar

**Wireframe:**
```
┌──────────────────────────────────────────────────────────┐
│ ▶️ 02:34:15 │ [Projeto: Website] [Atividade: Frontend]  │
│ [⏸️ Pausar] [⏹️ Parar] [🔄 Reiniciar]                    │
└──────────────────────────────────────────────────────────┘
```

**Componentes Necessários:**
- `Button` (shadcn/ui)
- `Select` (shadcn/ui) para projeto/atividade
- `Badge` (shadcn/ui) para status
- `Popover` (shadcn/ui) para configuração rápida

**Código Base:**
```typescript
import { useState, useEffect } from 'react'
import { Play, Pause, Square, RotateCcw } from 'lucide-react'
import { Button } from '@/components/ui/button'
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'
import { useTimerStore } from '@/stores/timerStore'
import { formatDuration } from 'date-fns'

export function TimerWidget() {
  const { isRunning, elapsedTime, startTimer, pauseTimer, stopTimer, resetTimer } = useTimerStore()
  const [selectedProject, setSelectedProject] = useState<string>('')
  const [selectedActivity, setSelectedActivity] = useState<string>('')

  // Atualizar tempo a cada segundo
  useEffect(() => {
    if (!isRunning) return
    const interval = setInterval(() => {
      // Lógica de incremento
    }, 1000)
    return () => clearInterval(interval)
  }, [isRunning])

  const handleStop = async () => {
    // Salvar registro no Supabase
    await stopTimer()
  }

  return (
    <div className="flex items-center gap-4">
      {/* Timer Display */}
      <div className="flex items-center gap-2">
        <span className="text-2xl font-mono font-bold">
          {formatDuration({ hours: 0, minutes: 0, seconds: elapsedTime })}
        </span>
      </div>

      {/* Seleção de Projeto/Atividade */}
      <Select value={selectedProject} onValueChange={setSelectedProject}>
        <SelectTrigger className="w-[200px]">
          <SelectValue placeholder="Selecione o projeto" />
        </SelectTrigger>
        <SelectContent>
          {/* Lista de projetos */}
        </SelectContent>
      </Select>

      <Select value={selectedActivity} onValueChange={setSelectedActivity}>
        <SelectTrigger className="w-[200px]">
          <SelectValue placeholder="Selecione a atividade" />
        </SelectTrigger>
        <SelectContent>
          {/* Lista de atividades */}
        </SelectContent>
      </Select>

      {/* Botões de Controle */}
      <div className="flex gap-2">
        {!isRunning ? (
          <Button onClick={startTimer} size="sm">
            <Play className="h-4 w-4 mr-2" />
            Iniciar
          </Button>
        ) : (
          <Button onClick={pauseTimer} size="sm" variant="secondary">
            <Pause className="h-4 w-4 mr-2" />
            Pausar
          </Button>
        )}

        <Button onClick={handleStop} size="sm" variant="destructive">
          <Square className="h-4 w-4 mr-2" />
          Parar
        </Button>

        <Button onClick={resetTimer} size="sm" variant="outline">
          <RotateCcw className="h-4 w-4" />
        </Button>
      </div>
    </div>
  )
}
```

**Store Zustand:**
```typescript
// /src/stores/timerStore.ts
import { create } from 'zustand'

interface TimerState {
  isRunning: boolean
  elapsedTime: number
  projectId: string | null
  activityId: string | null
  startTimer: () => void
  pauseTimer: () => void
  stopTimer: () => Promise<void>
  resetTimer: () => void
}

export const useTimerStore = create<TimerState>((set, get) => ({
  isRunning: false,
  elapsedTime: 0,
  projectId: null,
  activityId: null,

  startTimer: () => set({ isRunning: true }),
  pauseTimer: () => set({ isRunning: false }),
  
  stopTimer: async () => {
    const { elapsedTime, projectId, activityId } = get()
    
    // Salvar no Supabase
    // await supabase.from('contractor_timesheet_entries').insert({ ... })
    
    set({ isRunning: false, elapsedTime: 0, projectId: null, activityId: null })
  },
  
  resetTimer: () => set({ elapsedTime: 0 })
}))
```

---

### 3.3. Componente: `TimesheetTable.tsx`
**Localização:** `/src/components/timesheet/pj/TimesheetTable.tsx`

**Responsabilidades:**
- Exibir registros de horas em tabela
- Filtros avançados (período, projeto, status)
- Ações: Editar, Excluir, Duplicar, Submeter para Aprovação
- Paginação e ordenação

**Wireframe:**
```
┌────────────────────────────────────────────────────────────────┐
│ Filtros: [📅 Período: Este Mês ▼] [📁 Projeto: Todos ▼]       │
│          [🏷️ Status: Todos ▼] [🔍 Buscar...]                  │
├────────────────────────────────────────────────────────────────┤
│ ☑️ │ Data       │ Projeto    │ Atividade  │ Duração │ Valor  │ Ações │
├────┼────────────┼────────────┼────────────┼─────────┼────────┼───────┤
│ ☑️ │ 01/02/2026 │ Website    │ Frontend   │ 2h 30m  │ R$ 250 │ ⋮     │
│ ☐  │ 31/01/2026 │ App Mobile │ Backend    │ 4h 00m  │ R$ 400 │ ⋮     │
│ ☐  │ 30/01/2026 │ Website    │ Design     │ 1h 15m  │ R$ 125 │ ⋮     │
└────────────────────────────────────────────────────────────────┘
│ Total: 7h 45m │ R$ 775,00 │ [Submeter Selecionados]          │
└────────────────────────────────────────────────────────────────┘
```

**Componentes Necessários:**
- `Table` (shadcn/ui)
- `Select` (shadcn/ui) para filtros
- `Input` (shadcn/ui) para busca
- `Checkbox` (shadcn/ui) para seleção múltipla
- `DropdownMenu` (shadcn/ui) para ações
- `Badge` (shadcn/ui) para status
- `Pagination` (shadcn/ui)

**Código Base:**
```typescript
import { useState } from 'react'
import { useQuery } from '@tanstack/react-query'
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table'
import { Checkbox } from '@/components/ui/checkbox'
import { Badge } from '@/components/ui/badge'
import { Button } from '@/components/ui/button'
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger } from '@/components/ui/dropdown-menu'
import { MoreVertical, Edit, Trash, Copy } from 'lucide-react'
import { formatDuration, format } from 'date-fns'
import { ptBR } from 'date-fns/locale'

export function TimesheetTable() {
  const [selectedIds, setSelectedIds] = useState<string[]>([])
  const [filters, setFilters] = useState({
    period: 'this_month',
    projectId: null,
    status: 'all'
  })

  // Buscar registros do Supabase
  const { data: entries, isLoading } = useQuery({
    queryKey: ['timesheet-entries', filters],
    queryFn: async () => {
      // Lógica de busca no Supabase
      return []
    }
  })

  const handleSelectAll = (checked: boolean) => {
    if (checked) {
      setSelectedIds(entries?.map(e => e.id) || [])
    } else {
      setSelectedIds([])
    }
  }

  const handleSelectRow = (id: string, checked: boolean) => {
    if (checked) {
      setSelectedIds([...selectedIds, id])
    } else {
      setSelectedIds(selectedIds.filter(i => i !== id))
    }
  }

  const handleSubmitSelected = async () => {
    // Submeter para aprovação
  }

  return (
    <div className="space-y-4">
      {/* Filtros */}
      <div className="flex gap-4">
        {/* Componentes de filtro */}
      </div>

      {/* Tabela */}
      <div className="rounded-md border">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead className="w-12">
                <Checkbox
                  checked={selectedIds.length === entries?.length}
                  onCheckedChange={handleSelectAll}
                />
              </TableHead>
              <TableHead>Data</TableHead>
              <TableHead>Projeto</TableHead>
              <TableHead>Atividade</TableHead>
              <TableHead>Duração</TableHead>
              <TableHead>Valor</TableHead>
              <TableHead>Status</TableHead>
              <TableHead className="w-12"></TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {entries?.map((entry) => (
              <TableRow key={entry.id}>
                <TableCell>
                  <Checkbox
                    checked={selectedIds.includes(entry.id)}
                    onCheckedChange={(checked) => handleSelectRow(entry.id, checked as boolean)}
                  />
                </TableCell>
                <TableCell>
                  {format(new Date(entry.date), 'dd/MM/yyyy', { locale: ptBR })}
                </TableCell>
                <TableCell>{entry.project.name}</TableCell>
                <TableCell>{entry.activity.name}</TableCell>
                <TableCell>
                  {formatDuration({ hours: Math.floor(entry.duration / 3600), minutes: Math.floor((entry.duration % 3600) / 60) })}
                </TableCell>
                <TableCell>
                  {new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(entry.value)}
                </TableCell>
                <TableCell>
                  <Badge variant={entry.status === 'approved' ? 'success' : 'secondary'}>
                    {entry.status}
                  </Badge>
                </TableCell>
                <TableCell>
                  <DropdownMenu>
                    <DropdownMenuTrigger asChild>
                      <Button variant="ghost" size="sm">
                        <MoreVertical className="h-4 w-4" />
                      </Button>
                    </DropdownMenuTrigger>
                    <DropdownMenuContent align="end">
                      <DropdownMenuItem>
                        <Edit className="h-4 w-4 mr-2" />
                        Editar
                      </DropdownMenuItem>
                      <DropdownMenuItem>
                        <Copy className="h-4 w-4 mr-2" />
                        Duplicar
                      </DropdownMenuItem>
                      <DropdownMenuItem className="text-destructive">
                        <Trash className="h-4 w-4 mr-2" />
                        Excluir
                      </DropdownMenuItem>
                    </DropdownMenuContent>
                  </DropdownMenu>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>

      {/* Footer com Total e Ações */}
      <div className="flex items-center justify-between">
        <div className="text-sm text-muted-foreground">
          Total: {/* Cálculo de total */}
        </div>
        <Button
          onClick={handleSubmitSelected}
          disabled={selectedIds.length === 0}
        >
          Submeter Selecionados para Aprovação
        </Button>
      </div>
    </div>
  )
}
```

---

### 3.4. Componente: `TimesheetModal.tsx`
**Localização:** `/src/components/timesheet/pj/TimesheetModal.tsx`

**Responsabilidades:**
- Modal de criação/edição de registro
- Formulário com validação (React Hook Form + Zod)
- Campos: Projeto, Atividade, Data, Hora Início, Hora Fim, Descrição, Faturável

**Wireframe:**
```
┌────────────────────────────────────────────────────────┐
│ ✖️ Novo Registro de Horas                              │
├────────────────────────────────────────────────────────┤
│ Projeto *                                              │
│ [Selecione o projeto ▼]                                │
│                                                        │
│ Atividade *                                            │
│ [Selecione a atividade ▼]                              │
│                                                        │
│ Data *                    │ Hora Início * │ Hora Fim * │
│ [📅 01/02/2026]           │ [09:00]       │ [11:30]    │
│                                                        │
│ Descrição                                              │
│ [Desenvolvimento da tela de login...]                  │
│                                                        │
│ ☑️ Faturável                                           │
│                                                        │
│ Duração Calculada: 2h 30m │ Valor: R$ 250,00          │
├────────────────────────────────────────────────────────┤
│                          [Cancelar] [Salvar]           │
└────────────────────────────────────────────────────────┘
```

**Componentes Necessários:**
- `Dialog` (shadcn/ui)
- `Form` (shadcn/ui + React Hook Form)
- `Select` (shadcn/ui)
- `Calendar` + `Popover` (shadcn/ui) para seleção de data
- `Input` (shadcn/ui) para horários
- `Textarea` (shadcn/ui) para descrição
- `Checkbox` (shadcn/ui) para "Faturável"

**Código Base:**
```typescript
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import * as z from 'zod'
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter } from '@/components/ui/dialog'
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form'
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'
import { Input } from '@/components/ui/input'
import { Textarea } from '@/components/ui/textarea'
import { Checkbox } from '@/components/ui/checkbox'
import { Button } from '@/components/ui/button'
import { Calendar } from '@/components/ui/calendar'
import { Popover, PopoverContent, PopoverTrigger } from '@/components/ui/popover'
import { CalendarIcon } from 'lucide-react'
import { format } from 'date-fns'
import { ptBR } from 'date-fns/locale'

const formSchema = z.object({
  projectId: z.string().min(1, 'Projeto é obrigatório'),
  activityId: z.string().min(1, 'Atividade é obrigatória'),
  date: z.date({ required_error: 'Data é obrigatória' }),
  startTime: z.string().regex(/^([0-1]?[0-9]|2[0-3]):[0-5][0-9]$/, 'Formato inválido (HH:MM)'),
  endTime: z.string().regex(/^([0-1]?[0-9]|2[0-3]):[0-5][0-9]$/, 'Formato inválido (HH:MM)'),
  description: z.string().optional(),
  isBillable: z.boolean().default(true)
})

interface TimesheetModalProps {
  open: boolean
  onOpenChange: (open: boolean) => void
  entry?: any // Entry existente para edição
}

export function TimesheetModal({ open, onOpenChange, entry }: TimesheetModalProps) {
  const form = useForm<z.infer<typeof formSchema>>({
    resolver: zodResolver(formSchema),
    defaultValues: entry || {
      projectId: '',
      activityId: '',
      date: new Date(),
      startTime: '',
      endTime: '',
      description: '',
      isBillable: true
    }
  })

  const onSubmit = async (values: z.infer<typeof formSchema>) => {
    // Salvar no Supabase
    console.log(values)
    onOpenChange(false)
  }

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="sm:max-w-[600px]">
        <DialogHeader>
          <DialogTitle>
            {entry ? 'Editar Registro' : 'Novo Registro de Horas'}
          </DialogTitle>
        </DialogHeader>

        <Form {...form}>
          <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
            {/* Campo Projeto */}
            <FormField
              control={form.control}
              name="projectId"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Projeto *</FormLabel>
                  <Select onValueChange={field.onChange} defaultValue={field.value}>
                    <FormControl>
                      <SelectTrigger>
                        <SelectValue placeholder="Selecione o projeto" />
                      </SelectTrigger>
                    </FormControl>
                    <SelectContent>
                      {/* Lista de projetos */}
                    </SelectContent>
                  </Select>
                  <FormMessage />
                </FormItem>
              )}
            />

            {/* Campo Atividade */}
            <FormField
              control={form.control}
              name="activityId"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Atividade *</FormLabel>
                  <Select onValueChange={field.onChange} defaultValue={field.value}>
                    <FormControl>
                      <SelectTrigger>
                        <SelectValue placeholder="Selecione a atividade" />
                      </SelectTrigger>
                    </FormControl>
                    <SelectContent>
                      {/* Lista de atividades */}
                    </SelectContent>
                  </Select>
                  <FormMessage />
                </FormItem>
              )}
            />

            {/* Campos Data e Horários */}
            <div className="grid grid-cols-3 gap-4">
              <FormField
                control={form.control}
                name="date"
                render={({ field }) => (
                  <FormItem className="flex flex-col">
                    <FormLabel>Data *</FormLabel>
                    <Popover>
                      <PopoverTrigger asChild>
                        <FormControl>
                          <Button variant="outline" className="pl-3 text-left font-normal">
                            {field.value ? (
                              format(field.value, 'dd/MM/yyyy', { locale: ptBR })
                            ) : (
                              <span>Selecione</span>
                            )}
                            <CalendarIcon className="ml-auto h-4 w-4 opacity-50" />
                          </Button>
                        </FormControl>
                      </PopoverTrigger>
                      <PopoverContent className="w-auto p-0" align="start">
                        <Calendar
                          mode="single"
                          selected={field.value}
                          onSelect={field.onChange}
                          locale={ptBR}
                        />
                      </PopoverContent>
                    </Popover>
                    <FormMessage />
                  </FormItem>
                )}
              />

              <FormField
                control={form.control}
                name="startTime"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Hora Início *</FormLabel>
                    <FormControl>
                      <Input type="time" {...field} />
                    </FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />

              <FormField
                control={form.control}
                name="endTime"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Hora Fim *</FormLabel>
                    <FormControl>
                      <Input type="time" {...field} />
                    </FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />
            </div>

            {/* Campo Descrição */}
            <FormField
              control={form.control}
              name="description"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Descrição</FormLabel>
                  <FormControl>
                    <Textarea
                      placeholder="Descreva o que foi realizado..."
                      className="resize-none"
                      {...field}
                    />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />

            {/* Checkbox Faturável */}
            <FormField
              control={form.control}
              name="isBillable"
              render={({ field }) => (
                <FormItem className="flex flex-row items-start space-x-3 space-y-0">
                  <FormControl>
                    <Checkbox
                      checked={field.value}
                      onCheckedChange={field.onChange}
                    />
                  </FormControl>
                  <div className="space-y-1 leading-none">
                    <FormLabel>Faturável</FormLabel>
                  </div>
                </FormItem>
              )}
            />

            {/* Footer com Duração e Valor */}
            <div className="flex items-center justify-between rounded-md bg-muted p-4">
              <div>
                <span className="text-sm text-muted-foreground">Duração Calculada:</span>
                <span className="ml-2 font-semibold">2h 30m</span>
              </div>
              <div>
                <span className="text-sm text-muted-foreground">Valor:</span>
                <span className="ml-2 font-semibold">R$ 250,00</span>
              </div>
            </div>

            <DialogFooter>
              <Button type="button" variant="outline" onClick={() => onOpenChange(false)}>
                Cancelar
              </Button>
              <Button type="submit">Salvar</Button>
            </DialogFooter>
          </form>
        </Form>
      </DialogContent>
    </Dialog>
  )
}
```

---

### 3.5. Componente: `TimesheetCalendar.tsx`
**Localização:** `/src/components/timesheet/pj/TimesheetCalendar.tsx`

**Responsabilidades:**
- Visualização de registros em calendário
- Alternar entre visualizações: Mês, Semana, Dia
- Clicar em um dia para adicionar registro
- Exibir resumo de horas por dia

**Wireframe:**
```
┌────────────────────────────────────────────────────────────────┐
│ [◀️ Anterior] Fevereiro 2026 [Próximo ▶️]                      │
│ [📅 Mês] [📅 Semana] [📅 Dia]                                  │
├────────────────────────────────────────────────────────────────┤
│ Dom │ Seg │ Ter │ Qua │ Qui │ Sex │ Sáb │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  1  │  2  │  3  │  4  │  5  │  6  │  7  │
│     │ 2h  │ 4h  │     │ 6h  │ 3h  │     │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  8  │  9  │ 10  │ 11  │ 12  │ 13  │ 14  │
│     │ 5h  │     │ 7h  │ 4h  │     │     │
└────────────────────────────────────────────────────────────────┘
```

**Biblioteca Recomendada:**
```bash
npm install react-big-calendar
npm install date-fns
```

**Código Base:**
```typescript
import { useState } from 'react'
import { Calendar, dateFnsLocalizer, View } from 'react-big-calendar'
import { format, parse, startOfWeek, getDay } from 'date-fns'
import { ptBR } from 'date-fns/locale'
import 'react-big-calendar/lib/css/react-big-calendar.css'
import { Button } from '@/components/ui/button'
import { ChevronLeft, ChevronRight } from 'lucide-react'

const locales = {
  'pt-BR': ptBR
}

const localizer = dateFnsLocalizer({
  format,
  parse,
  startOfWeek,
  getDay,
  locales
})

export function TimesheetCalendar() {
  const [view, setView] = useState<View>('month')
  const [date, setDate] = useState(new Date())

  // Buscar eventos (registros de horas)
  const events = [
    {
      id: '1',
      title: 'Website - Frontend (2h 30m)',
      start: new Date(2026, 1, 1, 9, 0),
      end: new Date(2026, 1, 1, 11, 30),
      resource: {
        project: 'Website',
        activity: 'Frontend',
        duration: 9000, // segundos
        value: 250
      }
    }
    // ... mais eventos
  ]

  return (
    <div className="space-y-4">
      {/* Controles de Navegação */}
      <div className="flex items-center justify-between">
        <div className="flex gap-2">
          <Button
            variant="outline"
            size="sm"
            onClick={() => setDate(new Date(date.getFullYear(), date.getMonth() - 1))}
          >
            <ChevronLeft className="h-4 w-4" />
          </Button>
          <Button variant="outline" size="sm">
            {format(date, 'MMMM yyyy', { locale: ptBR })}
          </Button>
          <Button
            variant="outline"
            size="sm"
            onClick={() => setDate(new Date(date.getFullYear(), date.getMonth() + 1))}
          >
            <ChevronRight className="h-4 w-4" />
          </Button>
        </div>

        <div className="flex gap-2">
          <Button
            variant={view === 'month' ? 'default' : 'outline'}
            size="sm"
            onClick={() => setView('month')}
          >
            Mês
          </Button>
          <Button
            variant={view === 'week' ? 'default' : 'outline'}
            size="sm"
            onClick={() => setView('week')}
          >
            Semana
          </Button>
          <Button
            variant={view === 'day' ? 'default' : 'outline'}
            size="sm"
            onClick={() => setView('day')}
          >
            Dia
          </Button>
        </div>
      </div>

      {/* Calendário */}
      <div className="rounded-md border bg-background p-4" style={{ height: 600 }}>
        <Calendar
          localizer={localizer}
          events={events}
          startAccessor="start"
          endAccessor="end"
          view={view}
          onView={setView}
          date={date}
          onNavigate={setDate}
          culture="pt-BR"
          messages={{
            next: 'Próximo',
            previous: 'Anterior',
            today: 'Hoje',
            month: 'Mês',
            week: 'Semana',
            day: 'Dia'
          }}
        />
      </div>
    </div>
  )
}
```

---

## 4. INTEGRAÇÃO COM SUPABASE

### 4.1. Queries e Mutations (React Query)
```typescript
// /src/hooks/useTimesheetQueries.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { supabase } from '@/lib/supabase'

export function useTimesheetEntries(filters: any) {
  return useQuery({
    queryKey: ['timesheet-entries', filters],
    queryFn: async () => {
      const { data, error } = await supabase
        .from('contractor_timesheet_entries')
        .select(`
          *,
          project:timesheet_projects(*),
          activity:activities(*)
        `)
        .eq('contractor_id', 'user-id') // Substituir pelo ID do usuário logado
        .order('date', { ascending: false })

      if (error) throw error
      return data
    }
  })
}

export function useCreateTimesheetEntry() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async (entry: any) => {
      const { data, error } = await supabase
        .from('contractor_timesheet_entries')
        .insert(entry)
        .select()
        .single()

      if (error) throw error
      return data
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['timesheet-entries'] })
    }
  })
}

export function useUpdateTimesheetEntry() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async ({ id, ...entry }: any) => {
      const { data, error } = await supabase
        .from('contractor_timesheet_entries')
        .update(entry)
        .eq('id', id)
        .select()
        .single()

      if (error) throw error
      return data
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['timesheet-entries'] })
    }
  })
}

export function useDeleteTimesheetEntry() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async (id: string) => {
      const { error } = await supabase
        .from('contractor_timesheet_entries')
        .delete()
        .eq('id', id)

      if (error) throw error
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['timesheet-entries'] })
    }
  })
}
```

---

## 5. ESTILIZAÇÃO E TEMA

### 5.1. Tailwind CSS (já configurado no Humanamente)
O projeto já utiliza Tailwind CSS com shadcn/ui. Manter a consistência visual:

**Paleta de Cores:**
- Primary: `hsl(var(--primary))` - Azul
- Secondary: `hsl(var(--secondary))` - Cinza
- Success: `hsl(var(--success))` - Verde
- Destructive: `hsl(var(--destructive))` - Vermelho
- Muted: `hsl(var(--muted))` - Cinza claro

**Tipografia:**
- Font Family: Inter (já configurada)
- Tamanhos: `text-sm`, `text-base`, `text-lg`, `text-xl`, `text-2xl`

### 5.2. Customização do React Big Calendar
```css
/* /src/styles/calendar.css */
.rbc-calendar {
  font-family: 'Inter', sans-serif;
}

.rbc-event {
  background-color: hsl(var(--primary));
  border-radius: 4px;
  padding: 2px 5px;
  font-size: 0.875rem;
}

.rbc-today {
  background-color: hsl(var(--primary) / 0.1);
}

.rbc-header {
  font-weight: 600;
  padding: 10px 0;
  border-bottom: 2px solid hsl(var(--border));
}
```

---

## 6. CHECKLIST DE IMPLEMENTAÇÃO

### Fase 1: Setup e Dependências (1 dia)
- [ ] Instalar dependências necessárias
- [ ] Configurar React Query
- [ ] Configurar Zustand para timer
- [ ] Adicionar componentes shadcn/ui faltantes

### Fase 2: Componentes Base (2-3 dias)
- [ ] Criar `TimesheetPJDashboard.tsx`
- [ ] Criar `TimerWidget.tsx` + store Zustand
- [ ] Criar `TimesheetTable.tsx`
- [ ] Criar `TimesheetModal.tsx`
- [ ] Criar `TimesheetFilters.tsx`

### Fase 3: Calendário (1-2 dias)
- [ ] Integrar React Big Calendar
- [ ] Criar `TimesheetCalendar.tsx`
- [ ] Customizar estilos do calendário
- [ ] Implementar navegação (mês/semana/dia)

### Fase 4: Integração Supabase (1-2 dias)
- [ ] Criar hooks com React Query
- [ ] Implementar queries (buscar registros)
- [ ] Implementar mutations (criar, editar, excluir)
- [ ] Testar fluxo completo

### Fase 5: Refinamentos (1 dia)
- [ ] Ajustar responsividade
- [ ] Adicionar loading states
- [ ] Adicionar error handling
- [ ] Testes de usabilidade

---

## 7. COMPONENTES ADICIONAIS RECOMENDADOS

### 7.1. Filtros Avançados
```bash
npx shadcn-ui@latest add command
```
- Usar `Command` (shadcn/ui) para busca rápida de projetos/atividades

### 7.2. Exportação de Relatórios
```bash
npm install jspdf jspdf-autotable
```
- Exportar registros em PDF

### 7.3. Notificações
```bash
npx shadcn-ui@latest add toast
```
- Feedback visual para ações (salvar, excluir, submeter)

---

## 8. CONSIDERAÇÕES FINAIS

### Performance
- Usar `React.memo` para componentes pesados (tabela, calendário)
- Implementar paginação server-side no Supabase
- Lazy loading para calendário

### Acessibilidade
- Todos os componentes shadcn/ui já são acessíveis
- Adicionar `aria-labels` em botões de ação
- Garantir navegação por teclado

### Responsividade
- Mobile-first com Tailwind CSS
- Ocultar colunas menos importantes em mobile
- Usar `Sheet` (shadcn/ui) para filtros em mobile

---

**Próximo Passo:** Implementar os componentes seguindo esta especificação técnica.
