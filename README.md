# Airline Passenger Satisfaction - Dev Container & Notebook ML

Projeto didático de Machine Learning focado em prever a satisfação de passageiros de companhias aéreas utilizando o dataset [Airline Passenger Satisfaction](https://www.kaggle.com/datasets/teejmahal20/airline-passenger-satisfaction) do Kaggle.

O desenvolvimento é estruturado 100% de forma interativa via Jupyter Notebook, rodando em um ambiente isolado com VS Code Dev Containers, execução rootless, Python 3.11-slim e gerenciamento de dependências com uv.

---

## Estrutura do Projeto

```
eEDB-002/
├── .devcontainer/
│   ├── devcontainer.json          # Configuração do Dev Container no VS Code
│   ├── Dockerfile                 # Imagem rootless com Python 3.11-slim e uv
│   └── pyproject.toml             # Dependências mínimas (gerenciadas pelo uv)
├── .gitignore                     # Ignora .venv, __pycache__ e caches locais
├── README.md                      # Guia de replicação e documentação técnica
└── notebooks/
    └── exploracao_didatica.ipynb  # Notebook com o ciclo completo de EDA e ML
```

---

## Como Abrir e Executar no VS Code (Dev Container)

### 1. Pré-requisitos
- [Docker](https://docs.docker.com/get-docker/) instalado e em execução.
- [VS Code](https://code.visualstudio.com/) instalado.
- Extensão oficial [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) instalada no VS Code.

### 2. Abrir no Container
1. Abra a pasta do projeto no VS Code:
   ```bash
   code .
   ```
2. Quando a notificação aparecer no canto inferior direito, clique em "Reopen in Container" (ou use `Ctrl+Shift+P` -> `Dev Containers: Reopen in Container`).
3. O VS Code construirá o container automaticamente sob usuário não-root `vscode` e sincronizará as dependências com `uv sync`.

---

## Executando o Jupyter Notebook

1. No painel lateral de arquivos do VS Code, abra:
   `notebooks/exploracao_didatica.ipynb`
2. No canto superior direito do editor do notebook, clique em "Select Kernel" -> "Python Environments".
3. Selecione o interpretador virtual configurado pelo Dev Container:
   ```
   /workspace/.devcontainer/.venv/bin/python
   ```
4. Execute as células em sequência (`Shift + Enter` ou "Run All").

---

## Fundamentos Teóricos e Decisões de Arquitetura

### 1. O que são Features e Target?
- **Target ($y$)**: É a variável alvo que queremos classificar: `satisfaction` ($1 = \text{satisfeito}$, $0 = \text{neutro/insatisfeito}$).
- **Features ($X$)**: São as 23 variáveis descritivas utilizadas como entrada para a tomada de decisão do modelo (idade, distância do voo, atrasos, tipo de viagem, classe de assento e notas de 1 a 5 para os serviços de bordo e digitais).

### 2. Multicolinearidade Perfeita e a "Dummy Variable Trap"
A multicolinearidade perfeita ocorre quando uma feature pode ser prevista exatamente como uma combinação linear de outras.
- Ao converter variáveis categóricas (como `Gender` ou `Class`) em colunas binárias via One-Hot Encoding, uma das categorias torna-se redundante (ex: se o passageiro não é mulher, ele é homem).
- **Por que isso é um problema?**: Em modelos lineares (como a Regressão Logística), a redundância perfeita torna a matriz $X^T X$ singular e não-invertível, impedindo uma solução numérica estável e distorcendo os coeficientes.
- **Solução adotada**: Uso de `drop_first=True` no `pd.get_dummies()`. Para $K$ categorias, mantemos $K-1$ colunas, tornando a categoria excluída a referência base (*baseline*).

### 3. O que são Folds e por que são Utilizados? (Validação Cruzada)
- **k-Fold Cross-Validation**: Divide os dados de treino em $k=5$ blocos (folds) de igual tamanho. O treinamento ocorre em 5 rodadas: em cada rodada, 4 folds (80%) treinam o estimador e o fold restante (20%) valida a performance.
- **Por que utilizar?**: Evita que a métrica dependa de uma partição sortuda dos dados e atesta a capacidade de generalização do modelo, prevenindo *overfitting*.
- **Stratified K-Fold**: Garante que cada um dos 5 folds preserve exatamente a mesma proporção da variável alvo original (~56.7% insatisfeitos e ~43.3% satisfeitos).
- **Sem Vazamento de Dados (Strict Featurization Ordering)**: O escalonamento (`StandardScaler`) é recalculado do zero dentro de cada fold (`fit_transform` no treino do fold e `transform` na validação).

### 4. Matriz de Confusão
Cruza as predições do modelo com os dados reais do conjunto de teste independente:
- **Verdadeiro Negativo (TN)**: Passageiros insatisfeitos classificados corretamente.
- **Verdadeiro Positivo (TP)**: Passageiros satisfeitos classificados corretamente.
- **Falso Positivo (FP - Erro Tipo I)**: O modelo previu que o cliente estava satisfeito, mas ele estava insatisfeito (risco crítico de *churn* sem ação preventiva).
- **Falso Negativo (FN - Erro Tipo II)**: O modelo previu que o cliente estava insatisfeito, mas ele estava satisfeito.

### 5. Métricas de Avaliação e o F1-Score
- **Acurácia (~87.5%)**: Porcentagem total de predições corretas ($(TP + TN) / \text{Total}$).
- **Precisão (Precision)**: $TP / (TP + FP)$. Mede a confiabilidade de quando o modelo diz que um cliente está satisfeito.
- **Revocação (Recall / Sensibilidade)**: $TP / (TP + FN)$. Mede a capacidade do modelo de capturar todos os clientes que estavam realmente satisfeitos.
- **F1-Score**: É a **média harmônica** entre Precisão e Recall:
  $$\text{F1-Score} = 2 \times \frac{\text{Precisão} \times \text{Recall}}{\text{Precisão} + \text{Recall}}$$
  A média harmônica penaliza severamente o modelo caso uma das métricas seja baixa, garantindo um equilíbrio real de performance (atingindo **0.86** para satisfeitos e **0.89** para insatisfeitos).
