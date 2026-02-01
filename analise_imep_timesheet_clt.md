# Análise do Timesheet CLT - IMEP (Atual)

**URL:** https://painelrastreio.imeplab.com/dashboard/timesheet  
**Usuário:** Stephenson (Registro de ponto - CLT)  
**Data:** 01/02/2026

## Layout Atual (CLT)

### Header da Página
- **Título:** "Planilha de Horas"
- **Subtítulo:** "Registro de ponto - CLT"

### Abas Principais
1. **Batidas** (ativa)
2. **Justificativas**

### Painel Esquerdo - Registro de Ponto
- **Data e Hora Atual:** "Domingo, 01 De Fevereiro" + "12:59:36" (relógio em tempo real)
- **Localização:** "Rua Antônio de Barros, Vila Gomes Cardim, Tatuapé, São Paulo..." (geolocalização)
- **Botão Principal:** "Registrar Entrada" (verde, destaque)

**Batidas de Hoje:**
- Entrada: --:--
- Saída Almoço: --:--
- Retorno Almoço: --:--
- Saída: --:--

### Painel Direito - Calendário e Resumo
**Navegação de Mês:**
- Setas < > para navegar entre meses
- "Fevereiro 2026" (centralizado)

**Abas de Visualização:**
- Calendário (ativa)
- Lista

**Cards de Resumo:**
- Horas Trabalhadas: 0h 00m
- Dias Trabalhados: 0
- Banco de Horas: +0h 00m (verde)

**Calendário Mensal:**
- Grid com dias da semana (Dom, Seg, Ter, Qua, Qui, Sex, Sáb)
- Dia atual destacado (1 de fevereiro com borda azul)
- Visualização mensal completa

## Observações
- Interface é para **CLT** (batida de ponto), não para PJ (apontamento de horas)
- Design limpo com Tailwind CSS
- Geolocalização integrada
- Calendário mensal com navegação
- **Falta a interface PJ** que precisa ser implementada baseada no Kimai

## Problema Identificado
O usuário Stephenson está vendo a interface CLT, mas deveria ver a interface PJ (apontamento de horas por projeto/atividade). Preciso fazer logout e tentar acessar com o usuário correto (fabrizio.castro@imepcorp.com.br).
