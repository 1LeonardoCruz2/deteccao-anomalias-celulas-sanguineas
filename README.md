# Blood Cell Anomaly Detection

Projeto de Machine Learning para deteccao de anomalias em celulas sanguineas, inspirado pelo paper **CytoDiffusion** publicado na *Nature Machine Intelligence* (2025).

## Sobre o Projeto

O objetivo e construir modelos capazes de identificar celulas sanguineas anormais a partir de features tabulares (morfologia, coloracao, exames CBC e dados de aquisicao). O dataset contem **5.880 registros** com **19 tipos celulares** e **36 features**.

Tres tarefas de classificacao sao realizadas:

| Tarefa | Descricao |
|--------|-----------|
| Classificacao Binaria | Normal vs Anomalia (5 modelos comparados) |
| Classificacao Multi-Classe | Identificacao dos 19 tipos celulares |
| Predicao por Doenca | Leucemia, Anemia, Infeccao, Sickle Cell, Artefact |

## Data Leakage

O dataset original inclui 3 colunas de scores gerados pelo proprio CytoDiffusion (`cytodiffusion_anomaly_score`, `cytodiffusion_classification_confidence`, `labeller_confidence_score`). Essas features **vazam a informacao do target**, ja que foram produzidas a partir do label de anomalia.

Ao incluir esses scores, todos os modelos atingem AUC ~1.0 — um resultado artificialmente perfeito. Por isso, **essas 3 colunas sao removidas** da modelagem principal. O notebook mostra ambos os cenarios para evidenciar o problema.

## Dataset

**Fonte:** [Blood Cell Anomaly Detection 2025](https://www.kaggle.com/datasets/alitaqishah/blood-cell-anomaly-detection-2025) — Kaggle (Ali Taqi Shah)

| Arquivo | Descricao |
|---------|-----------|
| `blood_cell_anomaly_detection.csv` | Dataset principal — 5.880 celulas x 36 features |
| `cell_type_reference.csv` | Referencia clinica para os 19 tipos celulares |
| `cytodiffusion_benchmark_scores.csv` | Metricas de benchmark publicadas no paper |

### Features utilizadas na modelagem (29)

| Grupo | Features |
|-------|----------|
| Morfologia (9) | diametro, area do nucleo, densidade da cromatina, razao citoplasma, circularidade, excentricidade, granularidade, lobularidade, suavidade da membrana |
| Dimensoes (2) | area celular (px), perimetro (px) |
| Cor (4) | media R, G, B, intensidade do corante (Wright-Giemsa) |
| CBC Clinico (7) | contagem WBC, RBC, hemoglobina, hematocrito, plaquetas, MCV, MCHC |
| Demografico (2) | faixa etaria, sexo |
| Aquisicao (5) | fonte do dataset, protocolo de coloracao, microscopio, magnificacao, resolucao |

> As 3 colunas de scores CytoDiffusion (anomaly score, classification confidence, labeller confidence) foram **excluidas** por data leakage.

### Tipos Celulares (19)

**Normais (7):** Neutrophil, Lymphocyte, Monocyte, Eosinophil, Basophil, Normal RBC, Platelet

**Anormais (12):**

| Categoria | Tipos |
|-----------|-------|
| Leucemia | Blast Cell, Prolymphocyte |
| Anemia | Elliptocyte, Schistocyte, Spherocyte, Target Cell |
| Sickle Cell | Sickle Cell |
| Infeccao | Hypersegmented Neutrophil, Toxic Granulation, Reactive Lymphocyte |
| Artefato | Smudge Cell, Artefact |

## Modelos Utilizados

| Modelo | Tipo | Tarefa |
|--------|------|--------|
| Logistic Regression | Linear | Binaria |
| Random Forest | Ensemble (Bagging) | Binaria, Multi-Classe, Doenca |
| Gradient Boosting | Ensemble (Boosting) | Binaria, Multi-Classe |
| KNN (k=7) | Instance-based | Binaria |
| SVM (RBF) | Kernel | Binaria |

## Benchmark — CytoDiffusion (Paper)

| Metrica | CytoDiffusion | Baseline (ViT) |
|---------|---------------|-----------------|
| Anomaly Detection AUC | 0.990 | 0.916 |
| Blast Cell Sensitivity | 0.905 | 0.281 |
| Domain Shift Accuracy | 0.854 | 0.738 |
| Expert Fooled Rate | 0.523 (aleatorio) | — |

> O paper utiliza **imagens** de celulas diretamente. Nossos modelos usam **features tabulares** extraidas, por isso a comparacao e apenas ilustrativa.

## Estrutura do Projeto

```
blood-cell-anomaly-detection/
|-- README.md
|-- blood_cell_anomaly_detection.ipynb   # Notebook principal
|-- cell_Blood/
|   |-- blood_cell_anomaly_detection.csv
|   |-- cell_type_reference.csv
|   |-- cytodiffusion_benchmark_scores.csv
|-- *.png                                # 15 graficos gerados pelo notebook
```

## Como Executar

### Pre-requisitos

```bash
pip install pandas numpy matplotlib seaborn scikit-learn
```

### Execucao

1. Clone o repositorio:
```bash
git clone https://github.com/Csleo/blood-cell-anomaly-detection.git
cd blood-cell-anomaly-detection
```

2. Abra o notebook:
```bash
jupyter notebook blood_cell_anomaly_detection.ipynb
```

3. Execute todas as celulas sequencialmente (**Kernel > Restart & Run All**)

## Sumario do Notebook

| Secao | Conteudo |
|-------|----------|
| 1 | Importacao de Bibliotecas |
| 2 | Carregamento dos Dados (3 CSVs) |
| 3 | Analise Exploratoria (EDA) |
| 3.1–3.2 | Visao geral, distribuicao dos 19 tipos celulares |
| 3.3 | Distribuicao por categoria de doenca |
| 3.4–3.5 | Features morfologicas por tipo celular |
| 3.6 | Matriz de correlacao |
| 3.7 | Distribuicao dos scores CytoDiffusion |
| 3.8 | Analise demografica (idade e sexo) |
| 4 | Pre-processamento (encoding, remocao de leakage, scaling) |
| 5 | Classificacao Binaria — 5 modelos, ROC, confusion matrix, cross-validation |
| 6 | Classificacao Multi-Classe — 19 tipos celulares |
| 7 | Predicao por Categoria de Doenca |
| 8 | Feature Importance (binario e multi-classe) |
| 9 | Comparacao com Benchmark CytoDiffusion |
| 10–11 | Resumo dos Resultados e Conclusoes |

## Graficos Gerados

O notebook salva 15 visualizacoes automaticamente em `.png`:

1. Distribuicao dos tipos celulares + Normal vs Anomalia
2. Distribuicao por categoria de doenca
3. Features morfologicas — boxplot Normal vs Anomalia
4. Features chave por tipo celular
5. Matriz de correlacao
6. Scores CytoDiffusion — histograma por classe
7. Analise demografica (idade e sexo)
8. Comparacao de modelos — grafico de barras
9. Curvas ROC
10. Matriz de confusao — binario
11. Matriz de confusao — multi-classe (19 tipos)
12. Matriz de confusao — predicao de doenca
13. Feature importance — binario
14. Feature importance — multi-classe
15. Comparacao com benchmark CytoDiffusion

## Referencias

- Deltadahl et al. (2025). *Deep Generative Classification of Blood Cell Morphology.* Nature Machine Intelligence. DOI: 10.1038/s42256-025-01122-7
- Naouali & El Othmani (2025). *AI-Driven Automated Blood Cell Anomaly Detection.* Journal of Imaging, 11(5), 157.

## Licenca

Projeto para fins educacionais e de estudo.
