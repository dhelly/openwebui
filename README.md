# Open WebUI com LiteLLM Proxy e PostgreSQL

Um ambiente Docker completo para executar uma interface web de IA (Open WebUI) com proxy de modelos (LiteLLM) e banco de dados PostgreSQL para gerenciamento de configurações.

## 🚀 Visão Geral

Este projeto configura três serviços principais:

1. **Open WebUI** - Interface web para interagir com modelos de IA
2. **LiteLLM Proxy** - Proxy unificado para múltiplos provedores de IA
3. **PostgreSQL** - Banco de dados para armazenar configurações do LiteLLM

## 📋 Pré-requisitos

- Docker Engine 20.10+ e Docker Compose 2.0+
- Chaves de API dos provedores de IA (OpenAI, Anthropic, Google Gemini)

## 🛠️ Configuração Rápida

### 1. Clone o repositório (se aplicável)

```bash
git clone http://www.github.com/dhelly/openwebui
cd openwebui
```

### 2. Configure as variáveis de ambiente

```bash
# Copie o arquivo de exemplo
cp .env.example .env

# Edite o arquivo .env com suas chaves
nano .env
```

Exemplo de `.env` configurado:

```bash
# SUAS CHAVES DE API DOS PROVEDORES
OPENAI_API_KEY=sk-proj-sua-chave-openai-aqui
ANTHROPIC_API_KEY=sk-ant-sua-chave-anthropic-aqui
GEMINI_API_KEY=sua-chave-gemini-aqui

# CHAVE MESTRA DO LITELLM (DEFINA UMA CHAVE SEGURA)
LITELLM_MASTER_KEY=sk-litellm-uma-chave-segura-e-longa

# CHAVE DE CRIPTOGRAFIA (32 caracteres)
LITELLM_SALT_KEY=uma_chave_de_32_caracteres_1234567890

# SENHA DO POSTGRES
POSTGRES_PASSWORD=senha_segura_postgres_123

# CONFIGURAÇÕES DO BANCO DE DADOS
POSTGRES_USER=litellm
POSTGRES_DB=litellm
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
```

### 3. Configure os modelos no LiteLLM

Edite `litellm_config.yaml` para ajustar os modelos desejados:

```yaml
model_list:
  - model_name: gpt-4o
    litellm_params:
      model: openai/gpt-4o
      api_key: os.environ/OPENAI_API_KEY

  - model_name: claude-3-5-sonnet
    litellm_params:
      model: anthropic/claude-3-5-sonnet-20240620
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gemini-1-5-pro
    litellm_params:
      model: gemini/gemini-1.5-pro-latest
      api_key: os.environ/GEMINI_API_KEY

general_settings:
  master_key: "${LITELLM_MASTER_KEY}"
  store_model_in_db: true
```

### 4. Inicie os serviços

```bash
# Construa e inicie os containers em segundo plano
docker compose up -d --build

# Verifique o status
docker compose ps

# Acompanhe os logs
docker compose logs -f
```

## 🌐 Acesse as Interfaces

### Open WebUI
- **URL**: http://localhost:3001
- **Configuração**: Use a chave mestra do LiteLLM como chave de API no Open WebUI

### LiteLLM Proxy API
- **URL**: http://localhost:4000
- **Documentação**: http://localhost:4000/docs
- **Health Check**: http://localhost:4000/health/liveliness

### PostgreSQL (Apenas desenvolvimento)
- **Porta**: 5432
- **Usuário**: litellm
- **Banco**: litellm

## 🔧 Comandos Úteis

```bash
# Iniciar serviços
docker compose up -d

# Parar serviços
docker compose down

# Parar e remover volumes (cuidado: perde dados)
docker compose down -v

# Reiniciar serviços
docker compose restart

# Ver logs
docker compose logs -f litellm
docker compose logs -f openwebui
docker compose logs -f postgres

# Acessar container
docker exec -it litellm bash
docker exec -it openwebui bash
docker exec -it litellm_db bash

# Verificar saúde dos serviços
docker compose ps
```

## 📁 Estrutura do Projeto

```
.
├── docker-compose.yml          # Configuração dos serviços Docker
├── .env.example               # Exemplo de variáveis de ambiente
├── .env                       # Suas variáveis de ambiente (não versionado)
├── litellm_config.yaml        # Configuração dos modelos do LiteLLM
└── README.md                  # Este arquivo
```

## 🔒 Segurança

### O que não versionar:
1. **`.env`** - Contém chaves secretas
2. **Dados dos volumes** - `open-webui-data` e `postgres-data`

### Práticas recomendadas:
1. Use senhas fortes para `POSTGRES_PASSWORD`
2. Gere uma chave mestra segura para `LITELLM_MASTER_KEY`
3. Não exponha a porta do PostgreSQL (5432) em produção
4. Considere usar um proxy reverso (Nginx/Caddy) em produção

## 🧪 Testando a Configuração

### Teste o LiteLLM:

```bash
# Verifique se o proxy está respondendo
curl http://localhost:4000/health/liveliness

# Teste um modelo (substitua pela sua chave mestra)
curl http://localhost:4000/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sua_chave_mestra_aqui" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {"role": "user", "content": "Olá! Como você está?"}
    ]
  }'
```

### Teste o Open WebUI:
1. Acesse http://localhost:3001
2. Configure a chave de API como a chave mestra do LiteLLM
3. Selecione um modelo disponível
4. Teste uma conversa


## 🔄 Atualização

```bash
# Pare os serviços
docker compose down

# Atualize as imagens
docker compose pull

# Reinicie os serviços
docker compose up -d
```

## 📝 Notas

1. O LiteLLM usa PostgreSQL para armazenar configurações quando `STORE_MODEL_IN_DB=True`
2. O Open WebUI se conecta ao LiteLLM como proxy de modelos
3. Todos os dados de conversa são armazenados localmente no volume `open-webui-data`
4. As configurações do LiteLLM são persistidas no volume `postgres-data`

## 🤝 Contribuindo

1. Faça um fork do projeto
2. Crie uma branch para sua feature
3. Commit suas mudanças
4. Push para a branch
5. Abra um Pull Request

## 📄 Licença

Este projeto é distribuído sob a licença MIT. Veja o arquivo LICENSE para mais detalhes.

## 🙏 Agradecimentos

- [Open WebUI](https://github.com/open-webui/open-webui)
- [LiteLLM](https://github.com/BerriAI/litellm)
- [PostgreSQL](https://www.postgresql.org/)

---

**Importante**: Este setup é para ambiente de desenvolvimento. Para produção, considere:
- Configurar SSL/TLS
- Implementar autenticação adicional
- Configurar backups regulares
- Usar um proxy reverso
- Monitorar recursos e logs

