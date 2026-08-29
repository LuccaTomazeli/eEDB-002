FROM python:3.11-slim

# Evita que o Python gere arquivos .pyc desnecessários e ativa flush imediato no terminal
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# Instala ferramentas básicas do sistema
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    git \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Cria usuário não-root 'vscode' com UID/GID 1000 para sincronizar permissões no Linux
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=1000

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME -s /bin/bash

# Instala o gerenciador de pacotes 'uv' a partir da imagem oficial
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv
COPY --from=ghcr.io/astral-sh/uv:latest /uvx /usr/local/bin/uvx

# Define o diretório de trabalho padrão
WORKDIR /workspace

# Ajusta permissões do diretório de trabalho
RUN chown -R $USERNAME:$USERNAME /workspace

# Alterna para o usuário não-root
USER $USERNAME

# Garante que os binários instalados no ambiente de usuário estejam no PATH
ENV PATH="/home/$USERNAME/.local/bin:$PATH"
