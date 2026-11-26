# Classificação de Candidatos a Pulsares (HTRU2)

Projeto final desenvolvido para a disciplina **Introdução ao Aprendizado de Máquina**, aplicando um pipeline completo de Ciência de Dados (preparação, análise exploratória, modelagem, otimização e avaliação) sobre a base **HTRU2 Pulsar**.

## Integrantes
* **Heitor Cabral Braz de Araujo**

---

## 1. Descrição do Problema
Pulsares são estrelas de nêutrons raras de rotação rápida cuja detecção por radiotelescópios é dificultada pelo enorme volume de interferências de radiofrequência (RFI) e ruído de fundo. O objetivo deste projeto é construir modelos de classificação binária capazes de distinguir **pulsares reais (`is_pulsar = 1`)** de **não-pulsares (`is_pulsar = 0`)** a partir de estatísticas extraídas de seus sinais eletromagnéticos.

## 2. Base de Dados Utilizada
* **Fonte:** `CMPINAM_Dataset_07_HTRU2_Pulsar` ([Repositório da Disciplina](https://github.com/evertonjs/cmpinam-introducao-aprendizado-maquina/tree/main/projeto/datasets/CMPINAM_Dataset_07_HTRU2_Pulsar)), derivada do levantamento *High Time Resolution Universe Survey* (radiotelescópio Parkes).
* **Particionamento:**
  * `data/train.csv`: **14.318 instâncias** (13.007 não-pulsares [90,84%] e 1.311 pulsares [9,16%]).
  * `data/test.csv`: **3.580 instâncias** (3.252 não-pulsares [90,84%] e 328 pulsares [9,16%]).
* **Atributos:** 8 variáveis numéricas contínuas referentes à média, desvio padrão, excesso de curtose e assimetria do **Perfil de Pulso Integrado** e da **Curva DM-SNR**.

## 3. Metodologia
1. **Preparação sem Vazamento de Dados (*Data Leakage*):** Inspeção de nulos, duplicatas e consistência física realizada exclusivamente no conjunto de treino. Outliers foram mantidos por representarem assinaturas físicas reais de pulsares, sendo tratados via `RobustScaler` e modelos baseados em árvores.
2. **Análise Exploratória (EDA):** Investigação guiada por 8 perguntas analíticas (incluindo relações bivariadas e multivariadas) acompanhadas de gráficos justificados (barras, histograma/KDE, boxplots, heatmap de Spearman e dispersão).
3. **Modelagem e Validação:** Validação cruzada estratificada (`StratifiedKFold`, 5 dobras, `random_state=42`) com encapsulamento em `Pipeline`. Foram comparados quatro modelos:
   * `DummyClassifier` (*baseline* majoritário)
   * `LogisticRegression` (com `RobustScaler` e `class_weight='balanced'`)
   * `RandomForestClassifier` (com `class_weight='balanced'`)
   * `HistGradientBoostingClassifier` (otimizado via `RandomizedSearchCV` + análise de *Permutation Importance*)

## 4. Principais Resultados (Conjunto de Teste)

| Modelo | F1-Score | Recall | Precision | PR-AUC |
| :--- | :---: | :---: | :---: | :---: |
| Dummy Classifier (Baseline) | 0,0000 | 0,0000 | 0,0000 | 0,0916 |
| Regressão Logística | 0,8436 | **0,9207** | 0,7784 | **0,9334** |
| Random Forest | **0,8896** | 0,8598 | **0,9216** | 0,9249 |
| HistGradientBoosting (Tunado) | 0,8732 | 0,9024 | 0,8457 | 0,9211 |

* O **Random Forest** alcançou o maior **F1-Score (0,8896)** e a maior **Precisão (92,16%)**, minimizando falsos alarmes.
* O **HistGradientBoosting** ofereceu o melhor equilíbrio operacional, recuperando **90,24% dos pulsares reais** (296 de 328) com **84,57% de precisão**, tendo o atributo `excess_kurtosis_integrated_profile` como principal fator discriminante.

## 5. Estrutura do Repositório e Instruções de Execução

```text
.
├── data/
│   ├── train.csv
│   └── test.csv
├── notebooks/
│   └── analise_htru2.ipynb
├── requirements.txt
└── README.md