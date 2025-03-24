# Repositório Containers

Este repositório é responsável por gerenciar os ambientes de **desenvolvimento**, **testes** e **produção** usando Docker Compose. Ele contém submódulos para os repositórios do frontend e backend, e é configurado para ser atualizado automaticamente sempre que houver alterações nas branches correspondentes do repositório do frontend.

---

## Estrutura do Repositório

- **Branches**:

  - `dev`: Ambiente de desenvolvimento.
  - `test`: Ambiente de testes.
  - `main` (ou `production`): Ambiente de produção.

- **Submódulos**:

  - `frontend`: Repositório do frontend.
  - `backend`: Repositório do backend.

- **Arquivos de Configuração**:

  - `docker-compose.yml`: Configuração principal do Docker Compose.
  - `docker-compose.dev.yml`: Configurações específicas para o ambiente de desenvolvimento.
  - `docker-compose.test.yml`: Configurações específicas para o ambiente de testes.
  - `docker-compose.production.yml`: Configurações específicas para o ambiente de produção.

---

## Funcionamento

### 1. **Atualização Automática**

Quando um commit é feito nas branches `dev`, `test` ou `main` do repositório do frontend, um evento `repository_dispatch` é enviado para este repositório (**Containers**). Isso aciona os workflows correspondentes, que atualizam o submódulo do frontend e fazem o deploy no servidor.

### 2. **Workflows**

Os workflows estão configurados no diretório `.github/workflows` e são responsáveis por:

- **Atualizar o submódulo do frontend** para a branch correspondente.
- **Fazer o deploy no servidor** usando Docker Compose.

#### Workflows Disponíveis:

- **Deploy Dev Environment**: Acionado pela branch `dev`.
- **Deploy Test Environment**: Acionado pela branch `test`.
- **Deploy Production Environment**: Acionado pela branch `main`.

### 3. **Docker Compose**

Cada ambiente tem seu próprio arquivo de override (`docker-compose.dev.yml`, `docker-compose.test.yml`, `docker-compose.production.yml`), que personaliza as configurações do `docker-compose.yml` principal (como portas, variáveis de ambiente, etc.).

---

## Como Usar

### 1. **Configuração Inicial**

1. Clone este repositório:
   ```bash
   git clone --recurse-submodules https://github.com/seu-usuario/Containers.git
   ```
2. Navegue até o diretório do repositório:
   ```bash
   cd Containers
   ```
3. Atualize os submódulos:
   ```bash
   git submodule update --init --recursive
   ```

### 2. **Configuração dos Ambientes**

#### Desenvolvimento:

- **Branch**: `dev`
- **Arquivo de override**: `docker-compose.dev.yml`
- **Diretório no servidor**: `/var/www/dev`

#### Testes:

- **Branch**: `test`
- **Arquivo de override**: `docker-compose.test.yml`
- **Diretório no servidor**: `/var/www/test`

#### Produção:

- **Branch**: `main`
- **Arquivo de override**: `docker-compose.production.yml`
- **Diretório no servidor**: `/var/www/production`

---

## Deploy Automático

Quando um commit é feito no repositório do frontend, o workflow correspondente no repositório Containers é acionado.

- O submódulo do frontend é atualizado para a branch correta.
- O Docker Compose é executado no servidor, usando o arquivo de override correspondente.

### **Workflows no Repositório do Frontend**

O repositório do frontend contém um workflow que envia um evento `repository_dispatch` para este repositório sempre que há um push nas branches `dev`, `test` ou `main`. Esse workflow está localizado em `.github/workflows/notify-containers.yml`.

Exemplo:

```yaml
name: Notify Containers Repository

on:
  push:
    branches:
      - dev
      - test
      - main

jobs:
  notify:
    runs-on: ubuntu-latest

    steps:
      - name: Trigger Containers workflow
        uses: peter-evans/repository-dispatch@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          repository: seu-usuario/Containers
          event-type: frontend-updated
          client-payload: '{"ref": "${{ github.ref }}", "branch": "${{ github.ref_name }}"}'
```

### **Workflows no Repositório Containers**

Cada ambiente tem seu próprio workflow no diretório `.github/workflows`:

1. **Deploy Dev Environment**

   - Acionado pela branch `dev`.
   - Usa o arquivo de override `docker-compose.dev.yml`.

2. **Deploy Test Environment**

   - Acionado pela branch `test`.
   - Usa o arquivo de override `docker-compose.test.yml`.

3. **Deploy Production Environment**

   - Acionado pela branch `main`.
   - Usa o arquivo de override `docker-compose.production.yml`.

---

## Configuração do Servidor

Certifique-se de que o servidor tenha:

- Docker e Docker Compose instalados.
- Acesso SSH configurado com as chaves corretas.
- Diretórios para cada ambiente:
  - `/var/www/dev`
  - `/var/www/test`
  - `/var/www/production`

### **Segredos do GitHub**

Os seguintes segredos devem ser configurados no repositório Containers:

- `SERVER_HOST`: Endereço do servidor.
- `SERVER_USER`: Usuário do servidor.
- `SERVER_SSH_KEY`: Chave SSH para acesso ao servidor.

---

## Exemplo de Uso

1. Faça um commit na branch `dev` do repositório do frontend.
2. O workflow `Notify Containers Repository` é acionado e envia um evento para o repositório Containers.
3. O workflow `Deploy Dev Environment` no repositório Containers é acionado.
4. O submódulo do frontend é atualizado para a branch `dev`.
5. O Docker Compose é executado no servidor, usando o arquivo `docker-compose.dev.yml`.

---

## Contribuição

Se você encontrar problemas ou tiver sugestões, abra uma issue ou envie um pull request.

## Licença

Este projeto está licenciado sob a **MIT License**.

