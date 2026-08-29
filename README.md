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
├── README.md                      # Guia de replicação e execução
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

### Etapas do Pipeline no Notebook:
1. **Ingestão via API**: Download direto do dataset `teejmahal20/airline-passenger-satisfaction` com `kagglehub` (sem necessidade de tokens manuais).
2. **EDA Visual**: Gráficos com `seaborn` comparando satisfação por classe de voo, tipo de viagem e notas de serviços.
3. **Pré-processamento**: Tratamento de nulos em atrasos de voo e One-Hot Encoding das variáveis categóricas sem vazamento de dados.
4. **Treinamento com 5-Fold CV**: `StandardScaler` e `LogisticRegression` com validação cruzada estratificada.
5. **Avaliação no Teste**: Métricas completas, matriz de confusão e acurácia de ~87.5% no conjunto de teste independente.
6. **Interpretabilidade e Recomendações**: Gráfico dos coeficientes $\beta$ identificando as alavancas que mais aumentam a satisfação do cliente (ex: Embarque Online e Wi-Fi).
