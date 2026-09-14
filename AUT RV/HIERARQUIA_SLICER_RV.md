# Slicer hierárquico – Coordenador → Líder → Polo

Objetivo: ao selecionar um **Coordenador** (ex.: Italo Fabio) no slicer, filtrar todo o universo dele (líderes, polos, assistentes, técnicos). Mesma lógica para Líder e Polo.

---

## Ideia da solução

1. **Tabela de dimensão** com **nomes originais** (como na FactOS): uma linha por combinação distinta (Coordenador, Líder, Polo).
2. **Chave** em FactOS (e na dimensão) para relacionar: assim o slicer filtra a dimensão e o filtro desce para FactOS (e, se ligada, FactCSAT).
3. No Power BI: **Slicer** usando essa tabela com **hierarquia** (Coordenador → Líder → Polo).

---

## Passo 1: Coluna calculada na FactOS (chave da hierarquia)

Na tabela **FactOS**, crie uma **coluna calculada**:

**Nome da coluna:** `KeyHierarquia`

**Fórmula:**

```dax
KeyHierarquia =
VAR Coord = COALESCE( TRIM( FactOS[Coordenador.Coordenador] ), "" )
VAR Lider = COALESCE( TRIM( FactOS[Líder] ), "" )
VAR Polo  = COALESCE( TRIM( FactOS[des_service_provider] ), "" )
RETURN Coord & "|" & Lider & "|" & Polo
```

Assim cada linha da FactOS tem uma chave única por (Coordenador, Líder, Polo).

---

## Passo 2: Tabela calculada para o Slicer (nomes originais)

Crie uma **nova tabela calculada** no Power BI.

**Nome da tabela:** `DimHierarquiaSlicer`

**Fórmula:**

```dax
DimHierarquiaSlicer =
VAR Base =
    DISTINCT(
        SELECTCOLUMNS(
            FactOS,
            "Coordenador", TRIM( FactOS[Coordenador.Coordenador] ),
            "Líder",       TRIM( FactOS[Líder] ),
            "Polo",        TRIM( FactOS[des_service_provider] )
        )
    )
VAR SemVazios =
    FILTER(
        Base,
        NOT ISBLANK( [Coordenador] ) && [Coordenador] <> ""
    )
VAR ComChave =
    ADDCOLUMNS(
        SemVazios,
        "KeyHierarquia",
            [Coordenador] & "|" & [Líder] & "|" & [Polo]
    )
RETURN ComChave
```

Colunas da tabela: **Coordenador**, **Líder**, **Polo**, **KeyHierarquia** (nomes originais, como na FactOS).

---

## Passo 3: Relacionamento

- **DimHierarquiaSlicer[KeyHierarquia]** (1) —— **FactOS[KeyHierarquia]** (N)
- Cardinalidade: um para muitos (FactOS aponta para DimHierarquiaSlicer).
- Direção de filtro: **única** (DimHierarquiaSlicer → FactOS).
- Deixe **ativo**.

Assim, ao filtrar DimHierarquiaSlicer (por exemplo no slicer), o filtro vai para FactOS.

---

## Passo 4: Hierarquia no modelo

Na **DimHierarquiaSlicer**:

1. Clique com o botão direito na tabela → **Nova hierarquia**.
2. Nome sugerido: **Hierarquia Gestão**.
3. Arraste para dentro da hierarquia, nesta ordem:
   - **Coordenador**
   - **Líder**
   - **Polo**

---

## Passo 5: Slicer no relatório

1. Inserir um **Slicer**.
2. Em **Campo**: escolha a **hierarquia** **DimHierarquiaSlicer → Hierarquia Gestão** (e não um campo solto).
3. No slicer, ative **Hierarchy** / **Mostrar hierarquia** (conforme sua versão do Power BI), para poder expandir Coordenador → Líder → Polo.
4. Ao selecionar um coordenador (ex.: Italo Fabio), apenas as linhas da FactOS daquele coordenador (e dos líderes/polos dele) permanecem; as medidas passam a considerar só esse universo.

---

## FactCSAT (opcional)

Se você quiser que o mesmo slicer filtre também a **FactCSAT**:

- Verifique se a FactCSAT tem colunas equivalentes (ex.: Coordenador, Líder, Polo/des_service_provider).
- Se tiver, crie em FactCSAT a coluna **KeyHierarquia** da mesma forma (Coordenador & "|" & Líder & "|" & Polo).
- Crie relacionamento **DimHierarquiaSlicer[KeyHierarquia]** (1) —— **FactCSAT[KeyHierarquia]** (N), filtro **único** da dimensão para o fato.

Se a FactCSAT não tiver Coordenador/Líder/Polo, dá para manter só o filtro na FactOS; as medidas de CSAT que usam Eixo_Colaboradores e TREATAS continuam funcionando, e o filtro de “universo” vem da FactOS quando houver relacionamento entre as duas (por Matricula, etc.).

---

## Resumo

| Onde | O quê |
|------|--------|
| **FactOS** | Coluna calculada `KeyHierarquia` = Coord \| Líder \| Polo |
| **Nova tabela** | `DimHierarquiaSlicer` com Coordenador, Líder, Polo, KeyHierarquia (distinct da FactOS) |
| **Relacionamento** | DimHierarquiaSlicer (1) → FactOS (N) por KeyHierarquia |
| **Hierarquia** | Na DimHierarquiaSlicer: Coordenador → Líder → Polo |
| **Slicer** | Campo = essa hierarquia; opção de mostrar hierarquia ativada |

Com isso, ao escolher “Italo Fabio” no slicer, você restringe todo o universo dele (líderes, polos, assistentes e técnicos) nos dados da FactOS e nas medidas que usam FactOS.
