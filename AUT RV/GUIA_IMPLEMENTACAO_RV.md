# 🚀 Guia de Implementação - Dashboard de RV

## 📋 Passo 1: Carregar as 3 Bases no Power BI

### 1.1. Criar consulta DimColaboradores (Base 01)

1. **Power BI** → **Obter Dados** → **Excel**
2. Selecionar arquivo: `Dados_RV_Janeiro_2026_1.xlsx`
3. Selecionar planilha: `01 - Colaboradores Elegiveis`
4. **Transformar Dados** (Editor do Power Query)

**Código M** (ver `CODIGOS_M_RV.txt` - seção 1):
- Limpar `Matricula` (Text.Trim)
- Remover linhas sem matrícula
- Adicionar coluna `Tipo_Colaborador` (Técnico, Assistente, Líder, Coordenador, Matriz/Admin)
- Remover duplicatas por `Matricula`

**Renomear consulta para:** `DimColaboradores`

---

### 1.2. Criar consulta FactOS (Base 03)

1. **Obter Dados** → **Excel**
2. Mesmo arquivo, planilha: `03 - Dados_Dentro_Fora_Ativação`
3. **Transformar Dados**

**Código M** (ver `CODIGOS_M_RV.txt` - seção 2):
- Normalizar `Matricula` (texto)
- Remover linhas sem matrícula
- Padronizar campos de **SLA** e **Ativação** (criar colunas `SLA_Dentro_Prazo` e `Ativacao_Flag` como 0/1)
  - Exemplo: se coluna é "Está no Prazo?" ou "Dentro do Prazo" → `SLA_Dentro_Prazo` = 1 se "Sim/Dentro", 0 se "Não/Fora"
  - Exemplo: se coluna é "Ativação" → `Ativacao_Flag` = 1 se "Sim", 0 se "Não"

**Renomear consulta para:** `FactOS`

---

### 1.3. Criar consulta FactCSAT (Base 04)

1. **Obter Dados** → **Excel**
2. Mesmo arquivo, planilha: `04 - Dados_CSAT`
3. **Transformar Dados**

**Código M** (ver `CODIGOS_M_RV.txt` - seção 3):
- Normalizar `Matricula` (texto)
- Garantir que `nota_pesquisa` é numérico (1 a 5)
- Remover linhas sem matrícula ou nota inválida
- Renomear colunas (ex.: `Coordenador.Coordenador` → `Coordenador`)

**Renomear consulta para:** `FactCSAT`

---

### 1.4. Fechar e Aplicar

**Fechar e Aplicar** todas as consultas.

---

## 🔗 Passo 2: Criar Relacionamentos

### 2.1. Verificar Modelo de Dados

No **Modelo de Dados**, você deve ter:
- `DimColaboradores` (tabela de dimensão)
- `FactOS` (tabela de fatos)
- `FactCSAT` (tabela de fatos)

### 2.2. Criar Relacionamentos

1. **`DimColaboradores[Matricula]` ↔ `FactOS[Matricula]`**
   - Cardinalidade: **1:N** (Um para Muitos)
   - Direção de filtro: **Ambos**
   - Ativo: **Sim**

2. **`DimColaboradores[Matricula]` ↔ `FactCSAT[Matricula]`**
   - Cardinalidade: **1:N** (Um para Muitos)
   - Direção de filtro: **Ambos**
   - Ativo: **Sim**

---

## 📊 Passo 3: Criar Medidas DAX

### 3.1. Preparação: Verificar Campos

**Antes de criar as medidas**, verifique os **nomes exatos** das colunas nas tabelas:

- `DimColaboradores[Cargo]` → valores: "Técnico", "Assistente", "Líder", "Coordenador", "Matriz"
- `DimColaboradores[Polo RH Site]` → nome do polo
- `FactOS[SLA_Dentro_Prazo]` → 0 ou 1
- `FactOS[Ativacao_Flag]` → 0 ou 1
- `FactOS[Polo]` → código/nome do polo
- `FactOS[Líder]` → nome do líder
- `FactOS[Coordenador]` → nome do coordenador

**⚠️ IMPORTANTE:** Ajuste os nomes das colunas no código DAX conforme sua base real!

---

### 3.2. Criar Medidas (em `DimColaboradores`)

#### Medida 1: CSAT Final

```dax
CSAT_Final = 
VAR CargoAtual = SELECTEDVALUE(DimColaboradores[Cargo])
VAR MatriculaAtual = SELECTEDVALUE(DimColaboradores[Matricula])
VAR PoloAtual = SELECTEDVALUE(DimColaboradores[Polo RH Site])
RETURN
    SWITCH(
        TRUE(),
        // TÉCNICO: média CSAT individual
        CargoAtual = "Técnico",
            CALCULATE(
                AVERAGE(FactCSAT[nota_pesquisa]),
                FactCSAT[Matricula] = MatriculaAtual
            ),
        
        // ASSISTENTE: herda do polo
        CargoAtual = "Assistente",
            [CSAT_Polo],  // Ver medida abaixo
        
        // LÍDER: média dos polos do líder
        CargoAtual = "Líder" || CargoAtual = "Lider",
            VAR LiderAtual = SELECTEDVALUE(DimColaboradores[Nome])
            VAR PolosDoLider = 
                CALCULATETABLE(
                    DISTINCT(FactOS[Polo]),
                    FactOS[Líder] = LiderAtual
                )
            VAR CSAT_Polos = 
                ADDCOLUMNS(
                    PolosDoLider,
                    "@CSAT", [CSAT_Polo]
                )
            RETURN
                AVERAGEX(CSAT_Polos, [@CSAT]),
        
        BLANK()
    )
```

#### Medida 2: CSAT Polo (auxiliar)

```dax
CSAT_Polo = 
VAR PoloAtual = SELECTEDVALUE(DimColaboradores[Polo RH Site])
VAR MatriculasTecnicos = 
    CALCULATETABLE(
        VALUES(DimColaboradores[Matricula]),
        DimColaboradores[Polo RH Site] = PoloAtual,
        DimColaboradores[Cargo] = "Técnico"
    )
VAR CSAT_Tecnicos = 
    ADDCOLUMNS(
        MatriculasTecnicos,
        "@CSAT", 
            CALCULATE(AVERAGE(FactCSAT[nota_pesquisa]))
    )
RETURN
    AVERAGEX(CSAT_Tecnicos, [@CSAT])
```

#### Medidas 3 e 4: SLA e Ativação (mesmo padrão)

Ver código completo em `MEDIDAS_DAX_RV.txt` (seções 2 e 3).

**⚠️ IMPORTANTE:** As medidas hierárquicas (Líder, Coordenador) precisam de ajustes conforme a estrutura real da sua base. Pode ser necessário usar colunas de lookup ou criar tabelas auxiliares de hierarquia.

---

## 🎨 Passo 4: Criar Visual de Matriz Hierárquica

### 4.1. Visual de Matriz

1. **Inserir** → **Matriz**
2. **Eixos (Linhas)**:
   - `DimColaboradores[Coordenador]` (ou campo que identifica coordenador)
   - `DimColaboradores[Líder]` (ou campo que identifica líder)
   - `DimColaboradores[Polo RH Site]`
   - `DimColaboradores[Nome]` (ou `Matricula`)
   - `DimColaboradores[Cargo]`

3. **Valores**:
   - `CSAT_Final`
   - `SLA_Dentro_Prazo_Final`
   - `Ativacao_Percentual_Final`

### 4.2. Formatação

- Formatar medidas como **Porcentagem** (para SLA e Ativação) e **Decimal** (para CSAT)
- Adicionar cores condicionais (ex.: verde para >= 90%, amarelo 70-89%, vermelho < 70%)

---

## ✅ Passo 5: Validar Resultados

### 5.1. Validação de Técnicos

- Filtre por um **Técnico** específico
- Compare `CSAT_Final` com média manual das notas CSAT dele
- Compare `SLA_Final` com contagem manual de OS dentro/fora do prazo

### 5.2. Validação de Assistentes

- Filtre por um **Assistente**
- Verifique se `CSAT_Final`, `SLA_Final` e `Ativacao_Final` são **iguais ao polo** dele

### 5.3. Validação de Líderes

- Filtre por um **Líder**
- Calcule manualmente a média dos polos dele
- Compare com os valores da medida

---

## 🔧 Ajustes Necessários

### Ajuste 1: Nomes de Colunas

Verifique e ajuste conforme sua base:
- `FactOS[Polo]` vs `FactOS[Codigo Polo]` vs `FactOS[Polo RH Site]`
- `FactOS[Líder]` vs `FactOS[Lider]`
- `FactOS[Coordenador]` vs `FactOS[Coordenador.Coordenador]`

### Ajuste 2: Hierarquia Líder/Coordenador

Se os nomes de **Líder** e **Coordenador** não estão diretamente nas Facts, você pode:

**Opção A:** Criar tabelas auxiliares de lookup:
- `DimLider` (Matricula, Nome, Coordenador)
- `DimCoordenador` (Nome, ...)

**Opção B:** Usar colunas calculadas na `DimColaboradores`:
- `Lider` = LOOKUPVALUE(FactOS[Líder], FactOS[Matricula], DimColaboradores[Matricula])
- `Coordenador` = LOOKUPVALUE(FactOS[Coordenador], FactOS[Matricula], DimColaboradores[Matricula])

---

## 📝 Próximos Passos

1. ✅ Carregar as 3 bases
2. ✅ Criar relacionamentos
3. ✅ Criar medidas base (nível técnico)
4. ✅ Criar medidas de polo
5. ✅ Implementar lógica de assistente (herança)
6. ✅ Implementar medidas hierárquicas (Líder, Coordenador)
7. ✅ Validar resultados

---

## ❓ Dúvidas Comuns

**Q: Como saber qual campo usar para identificar Líder e Coordenador?**  
R: Verifique nas Facts (`FactOS`, `FactCSAT`) quais colunas têm os nomes dos líderes/coordenadores. Use essas colunas nas medidas.

**Q: As medidas de Líder/Coordenador estão dando erro?**  
R: Provavelmente os campos de hierarquia não estão corretos. Verifique os nomes das colunas nas Facts e ajuste o código DAX.

**Q: Como calcular o Total Geral (Matriz/Admin)?**  
R: Use `ALL(DimColaboradores)` ou simplesmente remova o filtro de cargo. A medida deve retornar a média geral de todos os coordenadores ou técnicos.
