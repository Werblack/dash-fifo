# Estrutura do Dashboard de RV

## 📋 Resumo da Lógica

**Técnico** calcula seu próprio resultado → **Polo** é a média dos técnicos → **Assistente** herda o resultado do polo → **Líder** é a média dos polos → **Coordenador** é a média dos líderes → **Matriz** é o total geral.

---

## 🗂️ Estrutura das 3 Bases

### Base 01 - Colaboradores Elegíveis
**Campos principais:**
- `Matricula` (chave)
- `Nome`
- `Cargo` (Técnico, Assistente, Líder, Coordenador, Matriz)
- `Polo RH Site` (identifica o polo da pessoa)
- `Periodo`
- `DATA ADMISSÃO` / `DATA DEMISSÃO`

**Objetivo:** Tabela de dimensão de pessoas. Define quem será medido.

---

### Base 03 - Dados_Dentro_Fora_Ativação
**Campos principais:**
- `Matricula` (do técnico que fez a OS)
- `Coordenador`
- `Líder`
- `Polo`
- Status de **SLA** (Dentro do prazo / Fora do prazo)
- Status de **Ativação** (Sim / Não)
- Outros campos da OS (Data, ID, etc.)

**Objetivo:** Todas as OS. Base para calcular:
- % Dentro do prazo (SLA)
- % Ativação

---

### Base 04 - Dados_CSAT
**Campos principais:**
- `Matricula` (do técnico que fez a OS)
- `Coordenador.Coordenador`
- `Líder`
- `Codigo Polo`
- `nota_pesquisa` (nota CSAT: 1 a 5)
- Outros campos da resposta CSAT

**Objetivo:** Todas as respostas CSAT. Base para calcular:
- CSAT individual
- CSAT do polo
- CSAT do líder
- CSAT do coordenador

---

## 🔑 Chave do Projeto: MATRÍCULA

Todas as tabelas se relacionam por `Matricula`:
- **DimColaboradores[Matricula]** ↔ **FactOS[Matricula]** (1:N)
- **DimColaboradores[Matricula]** ↔ **FactCSAT[Matricula]** (1:N)

---

## 📊 Os 3 Indicadores

1. **CSAT**: Média das notas de pesquisa
2. **% Dentro do prazo (SLA)**: OS dentro do prazo / Total de OS
3. **% Ativação**: OS ativadas / Total de OS

---

## 🎯 Lógica de Cálculo por Cargo

### (A) TÉCNICOS - Resultado 100% Individual
- Calcula seus próprios indicadores baseado em suas OS (Base 03) e CSAT (Base 04)

### (B) ASSISTENTES - Resultado Herdado do Polo
- **CSAT do assistente = CSAT do polo**
- **SLA do assistente = SLA do polo**
- **Ativação do assistente = Ativação do polo**

### (C) MATRIZ/ADMIN - Resultado Geral do Projeto
- Média de todos os coordenadores (ou média geral de técnicos)

---

## 📐 Hierarquia de Cálculo

### Polo = Média dos Técnicos
- CSAT do polo = `AVERAGE(CSAT dos técnicos do polo)`
- SLA do polo = `AVERAGE(SLA dos técnicos do polo)`
- Ativação do polo = `AVERAGE(Ativação dos técnicos do polo)`

### Líder = Média dos Polos
- CSAT do líder = `AVERAGE(CSAT dos polos do líder)`
- SLA do líder = `AVERAGE(SLA dos polos do líder)`
- Ativação do líder = `AVERAGE(Ativação dos polos do líder)`

### Coordenador = Média dos Líderes
- CSAT do coordenador = `AVERAGE(CSAT dos líderes do coordenador)`
- SLA do coordenador = `AVERAGE(SLA dos líderes do coordenador)`
- Ativação do coordenador = `AVERAGE(Ativação dos líderes do coordenador)`

### Total Geral = Média dos Coordenadores
- Média final de todos os coordenadores

---

## 🎨 Visualização Hierárquica

Estrutura esperada no visual:
```
Coordenador
  → Líder
    → Polo
      → Técnico / Assistente
        → CSAT | % SLA | % Ativação
```

Cada nível é calculado pela média do nível imediatamente inferior.
