# Chatbot WhatsApp com n8n e WAHA

Este projeto implementa um sistema de chatbot para WhatsApp usando n8n para automação de workflows e WAHA (WhatsApp HTTP API) para integração com a API do WhatsApp.

## 🏗️ Arquitetura do Sistema

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   WhatsApp      │    │      WAHA       │    │      n8n        │
│                 │◄──►│                 │◄──►│                 │
│   (Usuários)    │    │  (API Bridge)   │    │ (Workflows)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │     Redis       │    │   PostgreSQL    │
                       │   (Cache)       │    │   (Database)    │
                       └─────────────────┘    └─────────────────┘
```

## 🚀 Componentes

### 1. **WAHA (WhatsApp HTTP API)**

- **Função**: Ponte entre WhatsApp e n8n
- **Porta**: 3000
- **Responsabilidades**:
  - Conectar com WhatsApp Web
  - Receber mensagens, arquivos e mídias
  - Enviar webhooks para n8n
  - Servir arquivos via HTTP

### 2. **n8n (Workflow Automation)**

- **Função**: Motor de automação e processamento
- **Porta**: 5678
- **Responsabilidades**:
  - Processar webhooks do WAHA
  - Executar workflows de chatbot
  - Integrar com APIs externas
  - Gerenciar lógica de negócio

### 3. **Redis (Cache)**

- **Função**: Cache e sessões
- **Porta**: 6379
- **Responsabilidades**:
  - Armazenar sessões temporárias
  - Cache de dados frequentes
  - Gerenciamento de estado

### 4. **PostgreSQL (Database)**

- **Função**: Banco de dados principal
- **Porta**: 5432
- **Responsabilidades**:
  - Armazenar dados persistentes
  - Histórico de conversas
  - Configurações do sistema

## 📋 Pré-requisitos

- Docker e Docker Compose
- Número de telefone com WhatsApp
- QR Code para conectar WhatsApp Web

## 🛠️ Instalação e Configuração

### 1. Clone e Execute

```bash
git clone [seu-repositorio]
cd chatbot
docker-compose up -d
```

### 2. Configurar WhatsApp

1. Acesse `http://localhost:3000`
2. Escaneie o QR Code com seu WhatsApp
3. Aguarde a conexão ser estabelecida

### 3. Configurar n8n

1. Acesse `http://localhost:5678`
2. Crie sua conta de administrador
3. Configure workflows para processar mensagens

## 🔧 Configurações Importantes

### URLs e Acessos

| Serviço    | URL Local             | URL Externa      | Descrição              |
| ---------- | --------------------- | ---------------- | ---------------------- |
| WAHA       | http://localhost:3000 | http://[IP]:3000 | Interface WhatsApp API |
| n8n        | http://localhost:5678 | http://[IP]:5678 | Editor de Workflows    |
| Redis      | localhost:6379        | [IP]:6379        | Cache (interno)        |
| PostgreSQL | localhost:5432        | [IP]:5432        | Database (interno)     |

### Variáveis de Ambiente Críticas

```yaml
# WAHA
WHATSAPP_HOOK_URL: http://host.docker.internal:5678/webhook/webhook
WHATSAPP_API_URL: http://[SEU_IP]:3000
WHATSAPP_WEBHOOK_URL: http://[SEU_IP]:3000

# n8n
WEBHOOK_URL: http://host.docker.internal:5678
N8N_HOST: host.docker.internal
```

## 🔄 Fluxo de Funcionamento

### 1. **Recepção de Mensagem**

```
Usuário WhatsApp → WAHA → Webhook → n8n → Processamento
```

### 2. **Processamento**

- n8n recebe webhook do WAHA
- Analisa conteúdo da mensagem
- Executa workflow correspondente
- Pode integrar com APIs externas
- Processa lógica de negócio

### 3. **Resposta**

```
n8n → WAHA → WhatsApp → Usuário
```

### 4. **Arquivos e Mídia**

- Arquivos são armazenados no volume `waha_media`
- URLs são geradas: `http://[IP]:3000/api/media/[arquivo]`
- n8n pode processar e enviar arquivos de volta

## 📁 Estrutura de Volumes

```
volumes/
├── redis_data/          # Dados do Redis
├── postgres_data/       # Database PostgreSQL
├── waha_sessions/       # Sessões do WhatsApp
├── waha_media/          # Arquivos recebidos
└── n8n_data/           # Workflows e configurações n8n
```

## 🔐 Segurança

### Desenvolvimento

- Senhas padrão em todas as conexões
- Acesso HTTP (não HTTPS)
- Portas expostas para debug

### Produção (Recomendações)

- [ ] Alterar senhas padrão
- [ ] Implementar HTTPS
- [ ] Configurar proxy reverso (nginx)
- [ ] Restringir acesso por IP
- [ ] Backup automático dos volumes
- [ ] Monitoramento e logs

## 🚨 Troubleshooting

### Problemas Comuns

**1. QR Code não aparece**

```bash
docker-compose logs waha
```

**2. Webhooks não chegam no n8n**

- Verificar se `WHATSAPP_HOOK_URL` está correto
- Confirmar se n8n está rodando na porta 5678

**3. Arquivos não acessíveis**

- Substituir `host.docker.internal` pelo IP real
- Verificar se porta 3000 está aberta

**4. Conexão WhatsApp perdida**

- Reconectar via interface WAHA
- Verificar estabilidade da internet

### Logs Úteis

```bash
# Ver logs de todos os serviços
docker-compose logs

# Logs específicos
docker-compose logs waha
docker-compose logs n8n
docker-compose logs postgres
docker-compose logs redis
```

## 📈 Escalabilidade

### Para Alto Volume

- [ ] Implementar load balancer
- [ ] Usar Redis Cluster
- [ ] PostgreSQL com replicação
- [ ] Múltiplas instâncias WAHA
- [ ] Monitoramento (Prometheus/Grafana)

## 🔄 Backup e Restore

### Backup

```bash
# Backup dos volumes
docker run --rm -v chatbot_postgres_data:/data -v $(pwd):/backup ubuntu tar czf /backup/postgres_backup.tar.gz -C /data .
```

### Restore

```bash
# Restore dos volumes
docker run --rm -v chatbot_postgres_data:/data -v $(pwd):/backup ubuntu tar xzf /backup/postgres_backup.tar.gz -C /data
```

## 📞 Suporte

Para dúvidas e problemas:

1. Verifique os logs dos containers
2. Consulte a documentação oficial:
   - [WAHA Docs](https://github.com/devlikeapro/waha)
   - [n8n Docs](https://docs.n8n.io/)
3. Abra uma issue no repositório

---

**Versão**: 1.0.0  
**Última atualização**: $(date)  
**Compatibilidade**: Docker Compose v2.0+
