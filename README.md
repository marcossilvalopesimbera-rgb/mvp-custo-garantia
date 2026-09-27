# 📊 MVP Custo de Garantia — Imbera Brasil

> Projeto acadêmico desenvolvido na Pós-Graduação em **Data Science & Analytics** da **PUC-Rio**  
> Autor: Marcos Lopes Silva · RA: 4052024002499 · Jundiaí, SP  
> Empresa parceira: **Imbera Brasil** (FEMSA) — fabricante de refrigeração comercial

---

## 🎯 Visão geral

Este repositório reúne quatro MVPs desenvolvidos com **90.741 Ordens de Serviço reais** de garantia (Jan/2023–Abr/2026), cobrindo previsão de custos, segmentação de equipamentos, previsão de reincidência e engenharia de dados em nuvem.

Os MVPs foram projetados como um **sistema integrado** — não exercícios isolados:

```
MVP 2 (Clusters de falha)
     │
     ├──► Features → MVP 3 (Classificação de reincidência)
     │
     └──► Famílias problemáticas → MVP 1 (Previsão hierárquica)
                                        │
                                        └──► Camada Gold → MVP 4 (Pipeline nuvem)
```

---

## 📁 Estrutura do repositório

```
mvp-custo-garantia/
│
├── MVP_Previsao_Custo_Garantia.ipynb                          # MVP 1 — Séries temporais
├── MVP2_Segmentacao_Equipamentos.ipynb                        # MVP 2 — Clustering
├── MVP3_Reincidencia_Defeito.ipynb                            # MVP 3 — Classificação (em dev)
├── MVP4_Pipeline_Garantia_Final.ipynb                         # MVP 4 — Pipeline Databricks
│
├── MVP4_Engenharia_Dados_Marcos_Lopes_Silva_RA4052024002499_PUC_Rio.pdf  # Relatório MVP 4
│
├── base_garantia_MVP.xlsx                                     # Dataset original
├── base_garantia_MVP4_anonimizada.xlsx                        # Dataset MVP 4 (anonimizado)
└── README.md
```

---

## 🗂️ Dataset

| Atributo | Detalhe |
|---|---|
| **Volume** | 90.741 Ordens de Serviço |
| **Período** | Janeiro/2023 a Abril/2026 |
| **Famílias** | REFRIGERADOR e CONGELADOR |
| **Anonimização** | n° de série parcial + cliente codificado (CLI_XXXXX) |

**Feature de engenharia:** idade do equipamento extraída do n° de série (posições 3–6 = ano/mês de fabricação), gerando `idade_meses` e `ano_garantia` (1, 2 ou 3).

---

## 🧩 MVP 1 — Previsão de Custo de Garantia

**Arquivo:** `MVP_Previsao_Custo_Garantia.ipynb`  
**Disciplina:** Machine Learning & Analytics

**Problema:** Prever o custo total de garantia para H+1, H+2 e H+3 meses.

**Abordagem:**
- Modelos: Prophet, XGBoost, Ridge Regression, Random Forest, Moving Average (baseline)
- Validação: walk-forward (time series split)
- Métricas: MAE, RMSE, MAPE por horizonte

**Evolução planejada:** previsão hierárquica bottom-up por família de produto.

---

## 🧩 MVP 2 — Segmentação de Equipamentos por Perfil de Falha

**Arquivo:** `MVP2_Segmentacao_Equipamentos.ipynb`  
**Disciplina:** Machine Learning & Analytics

**Problema:** Identificar grupos de equipamentos com perfis de falha distintos.

**Abordagem:** K-Means (k=4), DBSCAN, PCA · Análise por modelo × ano de garantia

**Clusters identificados:**

| Cluster | Perfil | Característica |
|---|---|---|
| 0 | Manutenção frequente | Defeito em controlador |
| 1 | Desgaste normal | Problemas em LED |
| 2 | Equipamentos novos | LED, baixo custo |
| 3 | Casos críticos (37 OS) | Custo médio R$ 6.846/OS |

**Modelos problemáticos:** EVF19 (Ano 1), G3D26 (Ano 3), VRS13 (Ano 3)  
**Anomalias DBSCAN:** 2.409 casos para auditoria

---

## 🧩 MVP 3 — Previsão de Reincidência de Defeito *(em desenvolvimento)*

**Arquivo:** `MVP3_Reincidencia_Defeito.ipynb`  
**Disciplina:** Machine Learning & Analytics

**Problema:** Classificar se um equipamento voltará a abrir OS em até **90 dias**.

**Abordagem planejada:** XGBoost, Random Forest, Logistic Regression · Otimização de threshold · Features: família, modelo, idade, ano de garantia, defeito, cluster MVP 2

---

## 🧩 MVP 4 — Pipeline de Engenharia de Dados

**Arquivo:** `MVP4_Pipeline_Garantia_Final.ipynb`  
**Relatório:** `MVP4_Engenharia_Dados_Marcos_Lopes_Silva_RA4052024002499_PUC_Rio.pdf`  
**Disciplina:** Engenharia de Dados  
**Plataforma:** Databricks · Unity Catalog · Delta Lake

**Problema:** Transformar registros brutos de OS em base analítica estruturada na nuvem.

**Arquitetura Bronze → Silver → Gold:**

```
base_garantia_MVP4_anonimizada.xlsx
        │
        ▼
  Bronze (Volume Unity Catalog)     ← arquivo bruto na nuvem
        │
        ▼
  Silver (Delta Table)              ← limpeza, tipagem, feature engineering
  84.104 registros válidos
        │
        ▼
  Gold (Esquema Estrela)            ← fato_garantia + 4 dimensões
  Pronto para SparkSQL
```

**5 maiores insights:**

| Insight | Resultado |
|---|---|
| Custo cresceu 3× em 8 meses | Jan/23: R$114k → Ago/23: R$321k |
| REFRIGERADOR = 92,9% do custo | Pareto extremo — foco total nesta família |
| Mão de obra domina (52%) | Gargalo na rede técnica, não no componente |
| Controlador — 18.370 OS | Defeito mais frequente por larga margem |
| CLI_00013 = 88,2% do custo | Risco concentrado crítico |

**Hipóteses:** 4 confirmadas (2 superadas) · 1 parcialmente confirmada

---

## 🛠️ Stack técnico

| Área | Ferramentas |
|---|---|
| Linguagem | Python (Pandas, PySpark, NumPy, Scikit-learn, XGBoost, Prophet) |
| Plataforma nuvem | Databricks (Unity Catalog, Delta Lake, SparkSQL) |
| Machine Learning | Supervisionado, Não-supervisionado, Séries Temporais |
| Visualização | Matplotlib, Seaborn, Databricks built-in |

---

## ⚠️ Nota sobre os dados

Os dados são reais e de uso interno da Imbera Brasil, anonimizados antes da publicação:
- **n° de série:** preservados apenas os 4 caracteres de fabricação (ano/mês)
- **Cliente:** substituído por código sequencial (CLI_00001…) — consistência mantida para análises de concentração

Uso exclusivamente acadêmico no âmbito da PUC-Rio.

---

## 👤 Sobre o autor

**Marcos Lopes Silva** · Gestor industrial e cientista de dados aplicado · Jundiaí, SP

Mais de 15 anos liderando operações industriais — Engenharia de Processos, Manutenção e Qualidade. Este projeto é a expressão prática de uma convicção: as melhores decisões operacionais nascem quando gestão e dados falam a mesma língua.

> *"Não sou cientista de dados que aprendeu sobre fábricas lendo artigos. Sou gestor industrial que aprendeu a programar porque os dados da operação pediam isso."*

**Pós-Graduação em Data Science & Analytics — PUC-Rio** *(em curso)*

---

*Projeto desenvolvido como trabalho acadêmico — PUC-Rio, 2025/2026.*
