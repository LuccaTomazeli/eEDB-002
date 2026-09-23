# Airline Passenger Satisfaction - Regressao Logistica Pre-Voo

Projeto didatico de Machine Learning: prever se um passageiro ficara **satisfeito** com o voo usando apenas informacoes disponiveis **no momento da reserva**. O modelo e uma **Regressao Logistica** treinada no dataset [Airline Passenger Satisfaction](https://www.kaggle.com/datasets/teejmahal20/airline-passenger-satisfaction) do Kaggle.

Todo o conteudo esta em [notebooks/ML.ipynb](notebooks/ML.ipynb). Este README resume o que o notebook faz e como executa-lo.

---

## Estrutura do Projeto

```
eEDB-002/
├── .devcontainer/
│   ├── devcontainer.json   # Configuracao do Dev Container no VS Code
│   ├── Dockerfile          # Imagem rootless com Python 3.11-slim e uv
│   └── pyproject.toml      # Dependencias (gerenciadas pelo uv)
├── .gitignore
├── README.md
└── notebooks/
    └── ML.ipynb            # Notebook com o ciclo completo: EDA, modelo e avaliacao
```

---

## Como Executar

**Pre-requisitos:** [Docker](https://docs.docker.com/get-docker/), [VS Code](https://code.visualstudio.com/) e a extensao [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).

1. Abra a pasta do projeto no VS Code (`code .`) e clique em **Reopen in Container** (ou `Ctrl+Shift+P` -> `Dev Containers: Reopen in Container`). O container e construido com usuario nao-root e as dependencias sao instaladas com `uv sync`.
2. Abra `notebooks/ML.ipynb` e, em **Select Kernel** -> **Python Environments**, escolha:
   ```
   /workspace/.devcontainer/.venv/bin/python
   ```
3. Execute tudo com **Run All**. O dataset e baixado automaticamente via `kagglehub` na primeira execucao.

---

## Roteiro do Notebook

| Secao | O que acontece |
|---|---|
| 1. Configuracao | Imports e estilo dos graficos |
| 2. Dados | Carga das bases de treino (~104 mil) e teste (~26 mil) e definicao do alvo |
| 3. EDA | Qualidade dos dados, distribuicao do alvo, selecao de variaveis e efeito de confusao |
| 4. Pre-processamento | One-hot encoding e padronizacao |
| 5. Modelo | Treinamento e leitura dos coeficientes / Odds Ratios |
| 6. Avaliacao | ROC/AUC, matriz de confusao, F1, escolha do cutoff e tabela de decis |
| 7. Conclusao | Resumo dos aprendizados e aplicacao de negocio |

---

## Ideias-Chave

### O problema
A variavel resposta `satisfaction` e binaria:
- $Y = 1$: passageiro satisfeito (`satisfied`), ~43% da base
- $Y = 0$: passageiro neutro ou insatisfeito (`neutral or dissatisfied`)

### Sem vazamento de informacao (anti-leakage)
O dataset tem 22 variaveis explicativas, mas as 14 notas de servico de bordo e os atrasos **so existem depois do voo**. Usa-las deixaria o modelo otimo no papel e inutil na pratica. Por isso usamos apenas as 6 variaveis conhecidas na reserva:

`Gender`, `Customer Type`, `Age`, `Type of Travel`, `Class` e `Flight Distance`.

Pelo mesmo motivo, a EDA e a escolha do cutoff sao feitas **somente no treino**; o teste simula dados nunca vistos.

### Cuidado com o efeito de confusao
Isoladamente, passageiros satisfeitos parecem mais velhos e fazem voos mais longos. Mas esses perfis se concentram em **viagens de negocio na classe executiva**, o grupo mais satisfeito. A regressao estima os efeitos simultaneamente e mostra que, controlando por tipo de viagem e classe, `Age` e `Flight Distance` quase nao pesam.

### Regressao logistica e Odds Ratio
O modelo estima a probabilidade pela funcao sigmoide:

$$P(Y = 1 \mid X) = \frac{1}{1 + e^{-z}}, \qquad z = \beta_0 + \beta_1 X_1 + \dots + \beta_k X_k$$

Cada coeficiente vira um **Odds Ratio** ($OR = e^\beta$): acima de 1 aumenta a chance de satisfacao, abaixo de 1 reduz, perto de 1 nao altera.

| Variavel (vs. referencia) | OR | Leitura |
|---|---|---|
| Viagem pessoal (vs. negocio) | ~0,10 | chance ~90% menor |
| Cliente nao fiel (vs. fiel) | ~0,18 | chance ~82% menor |
| Eco Plus / Eco (vs. Business) | ~0,25 / ~0,28 | chance ~72-75% menor |
| Genero, idade, distancia | ~1,0 | efeito desprezivel |

### Avaliacao
O modelo gera um **score** (probabilidade de satisfacao), avaliado de duas formas:

- **Sem cutoff (ordenacao):** AUC de **~0,83** no treino e no teste, boa discriminacao e sem overfitting.
- **Com cutoff (decisao):** o corte padrao de 0,50 nao e obrigatorio. Escolhendo o cutoff que maximiza o F1 no treino (~0,27):

  | Cutoff | Acuracia | Precision | Recall | F1 |
  |---|---|---|---|---|
  | 0,50 | 0,78 | 0,78 | 0,69 | 0,73 |
  | 0,27 | 0,76 | 0,67 | 0,89 | 0,77 |

  E o *trade-off* classico: mais satisfeitos encontrados, ao custo de mais falsos positivos. O cutoff certo depende do custo de cada tipo de erro.

- **Tabela de decis:** ordenando os passageiros pelo score e dividindo em 10 grupos, os decis 1 a 4 tem ~68-84% de satisfeitos, enquanto os decis 7 a 10 ficam em ~10-15%. Ou seja, mesmo so com dados de reserva o modelo isola bem um grupo de **alto risco de insatisfacao**.

---

## Aplicacao e Limitacoes

**Na pratica:** antes do embarque, a companhia pode priorizar acoes preventivas (comunicacao proativa, upgrade, beneficios) nos passageiros de menor score.

**Limitacao:** com apenas 6 variaveis cadastrais o poder preditivo e limitado. As notas de servico explicariam muito mais, mas foram excluidas de proposito por so existirem depois do voo.

---

## Referencias
- Dataset: [Airline Passenger Satisfaction (Kaggle)](https://www.kaggle.com/datasets/teejmahal20/airline-passenger-satisfaction)
- [Statistics Fundamentals - Logistic Regression Examples](https://statisticsfundamentals.com/logistic-regression/logistic-regression-examples/)
