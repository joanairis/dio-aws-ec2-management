# 🔐 Práticas de Segurança em EC2

Guia essencial para manter suas instâncias EC2 seguras.

---

## 🚨 Princípios Fundamentais

### Princípio do Menor Privilégio

```
❌ ERRADO:
SSH (22): 0.0.0.0/0 ← Aberto para toda internet!

✅ CORRETO:
SSH (22): 203.0.113.45/32 ← Apenas seu IP
```

### Defesa em Profundidade

```
Camada 1: Security Groups (firewall)
    ↓
Camada 2: Network ACLs (subnet firewall)
    ↓
Camada 3: Firewall no SO (iptables/ufw)
    ↓
Camada 4: Autenticação em aplicação
    ↓
Camada 5: Criptografia de dados em trânsito
```

---

## 🔓 Grupos de Segurança

### Configuração Básica Segura

#### Web Server (HTTP/HTTPS)

```yaml
Entrada (Inbound):
  - SSH (22): De seu-ip/32
  - HTTP (80): De 0.0.0.0/0
  - HTTPS (443): De 0.0.0.0/0

Saída (Outbound):
  - Todos os protocolos: Para 0.0.0.0/0 (padrão)
```

#### Banco de Dados (Privado)

```yaml
Entrada (Inbound):
  - MySQL (3306): De sg-web-server (reference)
  - SSH (22): De seu-ip/32

Saída (Outbound):
  - Todos os protocolos: Para 0.0.0.0/0
```

#### API Backend

```yaml
Entrada (Inbound):
  - SSH (22): De seu-ip/32
  - HTTP (80): De sg-load-balancer (reference)
  - HTTPS (443): De sg-load-balancer (reference)
  - Custom TCP (3000): De sg-load-balancer

Saída (Outbound):
  - Todos os protocolos: Para 0.0.0.0/0
```

---

**Última atualização**: Junho 2026  
**Autora**: Joana Iris  
**Licença**: MIT