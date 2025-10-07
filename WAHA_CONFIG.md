# Configuração WAHA para URLs Externas

## Problema

O WAHA gera URLs com `localhost` que não são acessíveis externamente quando recebe arquivos.

## Soluções

### 1. Configuração via Variáveis de Ambiente (Recomendado)

Para que o WAHA gere URLs corretas, você precisa substituir `host.docker.internal` pelo IP real do seu servidor:

```bash
# Substitua [SEU_IP] pelo IP real do servidor
export WHATSAPP_API_URL=http://[SEU_IP]:3000
export WHATSAPP_WEBHOOK_URL=http://[SEU_IP]:3000
```

### 2. Configuração no docker-compose.yml

Se você souber o IP fixo do servidor, pode editar o docker-compose.yml:

```yaml
environment:
  WHATSAPP_API_URL: http://192.168.1.100:3000 # Substitua pelo seu IP
  WHATSAPP_WEBHOOK_URL: http://192.168.1.100:3000
```

### 3. Configuração via Proxy Reverso (Mais Robusta)

Adicione um nginx como proxy reverso:

```yaml
nginx:
  image: nginx:alpine
  ports:
    - "80:80"
    - "443:443"
  volumes:
    - ./nginx.conf:/etc/nginx/nginx.conf
  depends_on:
    - waha
    - n8n
```

### 4. Acesso aos Arquivos

Para acessar arquivos enviados via WhatsApp:

- **URL do arquivo**: `http://[SEU_IP]:3000/api/media/[filename]`
- **Exemplo**: `http://192.168.1.100:3000/api/media/document.pdf`

## Como Descobrir seu IP

```bash
# No macOS/Linux
ifconfig | grep "inet " | grep -v 127.0.0.1

# No Windows
ipconfig
```

## Teste da Configuração

1. Reinicie os containers:

```bash
docker-compose down
docker-compose up -d
```

2. Envie um arquivo via WhatsApp para o número conectado

3. Verifique se a URL no webhook do n8n aponta para o IP correto

## Notas Importantes

- Se o IP mudar frequentemente, considere usar um domínio dinâmico
- Para produção, use HTTPS com certificados SSL
- Configure firewall para permitir acesso à porta 3000
