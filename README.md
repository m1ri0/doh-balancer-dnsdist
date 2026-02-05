# DoH Load Balancer with DNSdist

Um servidor DNS over HTTPS (DoH) com balanceamento de carga usando DNSdist, cache inteligente e API REST para resolução de domínios.

## 📋 Visão Geral

Este projeto implementa uma solução completa de DNS over HTTPS (DoH) com as seguintes funcionalidades:

- **Balanceamento de carga** entre múltiplos servidores DNS upstream (Cloudflare e Google)
- **API REST** em Python/FastAPI para resolução de domínios
- **Cache DNS** com TTL configurável (até 24 horas)
- **DNS over TLS (DoT)** para comunicação com servidores upstream
- **Health checks** automáticos dos servidores upstream
- **Suporte HTTP/2** para melhor performance
- **Containerização** completa com Docker Compose

## 🏗️ Arquitetura

```
Cliente → API FastAPI (porta 8000) → DNSdist (porta 443) → Servidores Upstream
                                                              ├─ Cloudflare DoT (1.1.1.1:853)
                                                              ├─ Cloudflare DoT (1.0.0.1:853)
                                                              ├─ Google DoT (8.8.8.8:853)
                                                              └─ Google DoT (8.8.4.4:853)
```

### Componentes

1. **DNSdist**: Load balancer DNS com suporte a DoH/DoT
2. **API FastAPI**: Interface REST para consultas DNS
3. **Upstream Servers**: Cloudflare e Google DNS over TLS

## 🚀 Início Rápido

### Pré-requisitos

- Docker e Docker Compose
- Certificados SSL (`server.crt` e `server.key`) para HTTPS

### Instalação

1. Clone o repositório:
```bash
git clone <repository-url>
cd doh-balancer-dnsdist
```

2. Gere ou copie os certificados SSL:
```bash
# Exemplo: gerando certificado auto-assinado
openssl req -x509 -newkey rsa:4096 -keyout server.key -out server.crt -days 365 -nodes
```

3. Inicie os serviços:
```bash
docker-compose up -d
```

4. Verifique o status:
```bash
docker-compose ps
```

## 📡 Uso da API

### Endpoint de Resolução

**GET** `/resolve`

Resolve um domínio usando o balanceador DNS.

#### Parâmetros

- `url` (obrigatório): Domínio a ser resolvido
- `type` (opcional, padrão: "A"): Tipo de registro DNS (A, AAAA, MX, TXT, SOA, etc.)

#### Exemplos

```bash
# Resolver endereço IPv4
curl "http://localhost:8000/resolve?url=google.com"

# Resolver endereço IPv6
curl "http://localhost:8000/resolve?url=google.com&type=AAAA"

# Resolver registros MX
curl "http://localhost:8000/resolve?url=gmail.com&type=MX"

# Resolver registros TXT
curl "http://localhost:8000/resolve?url=google.com&type=TXT"
```

#### Resposta

```json
{
  "Answer": [
    {
      "name": "google.com.",
      "type": "A",
      "ttl": 300,
      "data": "142.250.185.46"
    }
  ]
}
```

## ⚙️ Configuração

### DNSdist

O arquivo [`dnsdist.conf`](dnsdist.conf) configura:

- **Servidores Upstream**: Cloudflare (peso 20) e Google (peso 10)
- **Política de Balanceamento**: `leastOutstanding` (menor número de consultas pendentes)
- **Cache**: 10.000 entradas, TTL máximo de 24 horas
- **Health Checks**: Verificação a cada 10 segundos

### Pesos de Balanceamento

Os pesos definem a distribuição de carga entre os servidores:

```lua
-- Cloudflare recebe 66% do tráfego (20+20)
-- Google recebe 33% do tráfego (10+10)
```

### Políticas de Balanceamento Disponíveis

- `leastOutstanding`: Envia para o servidor com menos consultas pendentes (padrão)
- `roundrobin`: Distribui igualmente entre todos os servidores
- `firstAvailable`: Usa o primeiro servidor disponível
- `whashed`: Hashing consistente baseado no domínio

## 📊 Monitoramento

### Healthcheck

A API possui healthcheck automático:

```bash
curl http://localhost:8000/resolve?url=google.com
```

### Logs do DNSdist

```bash
docker logs -f dnsdist_lb
```

### Logs da API

```bash
docker logs -f python_doh
```

## 🔧 Desenvolvimento

### Estrutura do Projeto

```
.
├── app/                    # API FastAPI
│   ├── main.py            # Aplicação principal
│   ├── requirements.txt   # Dependências Python
│   └── Dockerfile         # Container da API
├── doh-requester/         # Cliente de teste (opcional)
│   ├── main.py            # Script de benchmark
│   └── requirements.txt   # Dependências
├── dnsdist.conf           # Configuração do DNSdist
├── docker-compose.yml     # Orquestração dos serviços
└── README.md              # Este arquivo
```

### Dependências Python

```
dnspython       # Manipulação de mensagens DNS
httpx[http2]    # Cliente HTTP/2 assíncrono
fastapi         # Framework web
uvicorn         # Servidor ASGI
```

### Modificando a Configuração

Para alterar servidores upstream ou pesos, edite [`dnsdist.conf`](dnsdist.conf) e reinicie:

```bash
docker-compose restart dnsdist
```

## 🧪 Testes

### Teste de Resolução Básico

```bash
# Via API
curl "http://localhost:8000/resolve?url=example.com"

# Via DoH direto (HTTPS)
curl -H "accept: application/dns-json" \
     "https://localhost/dns-query?name=example.com&type=A"
```

### Benchmark (Opcional)

O módulo `doh-requester` pode ser usado para testes de carga:

```bash
# Descomentar no docker-compose.yml
docker-compose up requester
```

## 🔒 Segurança

- ✅ Comunicação criptografada (TLS) com servidores upstream
- ✅ Validação de certificados SSL
- ✅ Suporte a HTTPS para a API
- ⚠️ Para produção, configure `verify=True` no cliente httpx ([app/main.py](app/main.py#L15))
- ⚠️ Use certificados válidos (não auto-assinados) em produção

## 📝 Portas

| Serviço | Porta | Protocolo | Descrição |
|---------|-------|-----------|-----------|
| DNSdist | 53    | UDP/TCP   | DNS tradicional |
| DNSdist | 443   | TCP       | DNS over HTTPS |
| API     | 8000  | TCP       | REST API |

## 🛠️ Troubleshooting

### Erro de conexão com DNSdist

```bash
# Verifique se o DNSdist está rodando
docker-compose ps dnsdist

# Verifique os logs
docker logs dnsdist_lb
```

### Certificado SSL inválido

Certifique-se de que os arquivos `server.crt` e `server.key` existem e são válidos:

```bash
openssl x509 -in server.crt -text -noout
```

### Cache não está funcionando

Verifique as estatísticas de cache no DNSdist:

```bash
docker exec -it dnsdist_lb dnsdist -c
> showCacheHitResponseStats()
```

## 👥 Contribuição

Contribuições são bem-vindas! Por favor, abra uma issue ou pull request.

## 📚 Referências

- [DNSdist Documentation](https://dnsdist.org/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [DNS over HTTPS RFC 8484](https://datatracker.ietf.org/doc/html/rfc8484)
- [DNS over TLS RFC 7858](https://datatracker.ietf.org/doc/html/rfc7858)