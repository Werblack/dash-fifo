# Correções – MATRIZ no Eixo_Colaboradores

## Objetivo
- Matriz: Coordenadores, Líderes e Assistentes todos sob "Matriz" → Cargo → Nome
- Sem campos vazios
- Coordenadores e Líderes da Matriz incluídos
- Non-Matriz: hierarquia normal (Coordenador → Líder → Polo → Cargo → Nome)

---

## 1. Eixo_Colaboradores – alterações no _ops_dim

No bloco _ops_dim, altere a linha de "Polo Base (Regra)" para normalizar Matriz:

**De:**
```
"Polo Base (Regra)",   DimColaboradores[Polo_Base],
```

**Para:** Adicione __EhMatrizPolo no ADDCOLUMNS e use no SELECTCOLUMNS. O ADDCOLUMNS fica:

```
ADDCOLUMNS(
    DimColaboradores,
    "__CargoNorm", SUBSTITUTE( UPPER(TRIM(DimColaboradores[Cargo])), "Í","I" ),
    "__EhMatrizPolo", 
        VAR x = UPPER(TRIM(COALESCE(DimColaboradores[Polo_Base], "")))
        RETURN x = "MATRIZ" || x = "PAGRESOLVE MATRIZ SP"
),
```

E em SELECTCOLUMNS, Polo Base (Regra):
```
"Polo Base (Regra)",   IF([__EhMatrizPolo], "Matriz", DimColaboradores[Polo_Base]),
```

---

## 2. Eixo_Colaboradores – alterações em _gest_matriz_dim

No __EhMatriz, inclua "PAGRESOLVE MATRIZ SP":

**De:**
```
RETURN _txt || UPPER(TRIM(COALESCE(DimColaboradores[Polo_Base], ""))) = "MATRIZ"
```

**Para:**
```
VAR _polo = UPPER(TRIM(COALESCE(DimColaboradores[Polo_Base], "")))
RETURN _txt || _polo = "MATRIZ" || _polo = "PAGRESOLVE MATRIZ SP"
```

(Faça o mesmo em _dim_gestores_matriz_norm.)

---

## 3. Eixo_Colaboradores – _tecnicos_fatos_add (Polo Base para Matriz)

No _tecnicos_fatos_add, normalize Polo Base (Regra) para Matriz:

**De:**
```
"Polo Base (Regra)",   COALESCE([__PoloDIM], [__Polo03]),
```

**Para:** Envolva em ADDCOLUMNS para criar __PoloNorm e use:
```
"Polo Base (Regra)",   
    IF( UPPER(TRIM(COALESCE([__PoloDIM],[__Polo03],""))) IN {"MATRIZ","PAGRESOLVE MATRIZ SP"}, 
        "Matriz", 
        COALESCE([__PoloDIM], [__Polo03]) ),
```

---

## 4. Eixo_Colaboradores – código completo (referência)

O arquivo TABELAS_CALCULADAS_RV.txt foi atualizado com o Eixo_Colaboradores completo. As alterações principais:
- _ops_dim: Polo Base (Regra) = "Matriz" quando Polo_Base = Matriz ou PagResolve Matriz SP
- _gest_matriz_dim e _dim_gestores_matriz_norm: __EhMatriz inclui "PAGRESOLVE MATRIZ SP"
- _tecnicos_fatos_add: Polo Base (Regra) = "Matriz" quando __PoloDIM/__Polo03 indicam Matriz
- _dim_assistentes_mats: exclusão mantida (Assistentes não viram Técnicos)

---

## 5. Colunas calculadas Coordenador e Líder

**Coordenador:**
```
Coordenador =
VAR PoloAtual = [Polo Base (Regra)]
VAR PoloUpper = UPPER(TRIM(COALESCE(PoloAtual, "")))
VAR EhMatriz = [IsMatriz] || PoloUpper = "MATRIZ" || PoloUpper = "PAGRESOLVE MATRIZ SP"
RETURN
    IF(
        EhMatriz,
        "Matriz",
        IF(
            ISBLANK(PoloAtual) || PoloAtual = "",
            BLANK(),
            CALCULATE(
                FIRSTNONBLANK(FactOS[Coordenador.Coordenador], 1),
                FILTER(ALL(FactOS), FactOS[des_service_provider] = PoloAtual)
            )
        )
    )
```

**Líder:**
```
Líder =
VAR PoloAtual = [Polo Base (Regra)]
VAR PoloUpper = UPPER(TRIM(COALESCE(PoloAtual, "")))
VAR EhMatriz = [IsMatriz] || PoloUpper = "MATRIZ" || PoloUpper = "PAGRESOLVE MATRIZ SP"
RETURN
    IF(
        EhMatriz,
        "Matriz",
        IF(
            ISBLANK(PoloAtual) || PoloAtual = "" || PoloAtual = "Matriz",
            BLANK(),
            CALCULATE(
                FIRSTNONBLANK(FactOS[Líder], 1),
                FILTER(ALL(FactOS), FactOS[des_service_provider] = PoloAtual)
            )
        )
    )
```
