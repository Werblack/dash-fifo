# Mapeamento: Dados_RV_Janeiro_2026_1.xlsx → Dashboard RV

Este dashboard será uma **nova aba** do Dash de Justificativa de Abono.

---

## Arquivo base

| Arquivo | Caminho |
|--------|---------|
| **Dados_RV_Janeiro_2026_1.xlsx** | `AUT RV\RV_JANEIRO26\Dados_RV_Janeiro_2026_1.xlsx` |

---

## Abas do Excel → Tabelas no Power BI

| Aba no Excel | Consulta no Power BI | Uso |
|--------------|----------------------|-----|
| **01 - Colaboradores Elegiveis** | **DimColaboradores** | Dimensão de pessoas (Matricula, Nome, Cargo, Polo RH Site, Periodo, etc.) |
| **03 - Dados_Dentro_Fora_Ativação** | **FactOS** | Fatos de OS (SLA, Ativação, Matricula, Coordenador, Líder, Polo) |
| **04 - Dados_CSAT** | **FactCSAT** | Respostas CSAT (Matricula, nota_pesquisa, Líder, Coordenador, Codigo Polo) |

**Importante:** Se no seu `Dados_RV_Janeiro_2026_1.xlsx` as abas tiverem **nomes diferentes**, altere no Power Query (códigos M) a linha que referencia a aba, por exemplo:

- `Fonte{[Item="01 - Colaboradores Elegiveis", Kind="Sheet"]}[Data]`
- Troque `"01 - Colaboradores Elegiveis"` pelo nome exato da aba.

---

## Chave e relacionamentos

- **Chave:** `Matricula` (texto, sem espaços).
- **DimColaboradores[Matricula]** (1) ↔ **FactOS[Matricula]** (N)
- **DimColaboradores[Matricula]** (1) ↔ **FactCSAT[Matricula]** (N)

---

## Indicadores calculados

| Indicador | Fonte | Lógica por cargo |
|-----------|--------|-------------------|
| **CSAT** | FactCSAT (nota_pesquisa) | Técnico: individual; Assistente: herda polo; Líder/Coordenador: média dos subordinados |
| **% Dentro do prazo (SLA)** | FactOS (SLA_Dentro_Prazo) | Idem |
| **% Ativação** | FactOS (Ativacao_Flag) | Idem |

---

## Onde estão os códigos

- **Power Query (M):** `CODIGOS_M_RV.txt` (já apontando para `Dados_RV_Janeiro_2026_1.xlsx`)
- **Medidas DAX:** `MEDIDAS_DAX_RV.txt`
- **Estrutura e lógica:** `ESTRUTURA_DASH_RV.md`
- **Passo a passo:** `GUIA_IMPLEMENTACAO_RV.md`
