# 📚 Conceitos Fundamentais de EC2

## O que é EC2?

**EC2 (Elastic Compute Cloud)** é um serviço de computação em nuvem que fornece capacidade de processamento escalável. É como alugar um computador na nuvem, pagando apenas pelo tempo de uso.

## Vantagens do EC2

| Vantagem | Benefício |
|---------|----------|
| **Escalabilidade** | Aumentar ou reduzir recursos conforme necessário |
| **Flexibilidade** | Escolher entre diferentes configurações |
| **Confiabilidade** | 99,99% de uptime SLA |
| **Segurança** | Múltiplas camadas de proteção |
| **Custo-benefício** | Pague apenas o que usar |
| **Controle Total** | Acesso root às instâncias |

## Termos Importantes

### Instância
Uma máquina virtual que executa na nuvem AWS. Pode ser parada, iniciada, reiniciada ou encerrada.

### AMI (Amazon Machine Image)
Uma imagem pré-configurada contendo:
- Sistema operacional
- Software e aplicações
- Permissões e configurações

### VPC (Virtual Private Cloud)
Uma rede privada isolada onde sua instância roda. Oferece controle total sobre:
- Intervalo de IPs
- Subnets
- Tabelas de rota
- Gateways

### Grupo de Segurança
Firewall virtual que controla o tráfego de entrada e saída:
- Define regras por porta
- Define regras por protocolo
- Define quem pode acessar

### Elastic IP
Um endereço IP público estático que não muda quando a instância é parada.

### EBS (Elastic Block Store)
Armazenamento em bloco para instâncias EC2. Funciona como um disco rígido:
- Pode ser desanexado
- Pode ser fotografado (snapshot)
- Oferece diferentes tipos de performance

## Ciclo de Vida de uma Instância

```
┌─────────┐
│ pending │  (0-30 segundos)
└────┬────┘
     ↓
┌─────────┐
│ running │  (operacional)
└────┬────┘
     ↓
┌──────────┐
│ stopping │  (em processo de parada)
└────┬─────┘
     ↓
┌─────────┐
│ stopped │  (pausada, sem cobranças de computação)
└────┬────┘
     ↓
┌──────────────┐
│ terminating  │  (em processo de exclusão)
└────┬─────────┘
     ↓
┌──────────┐
│terminated│  (deletada permanentemente)
└──────────┘
```

## Estados Detalhados

### Pending
- Instância está sendo iniciada
- Duração típica: 0-30 segundos
- Não pode receber conexões ainda

### Running
- Instância está ativa e operacional
- Pode receber conexões
- Está sendo cobrada (computação + armazenamento)

### Stopping
- Processo de parada em andamento
- Duração típica: 1-3 minutos
- Estado transitório

### Stopped
- Instância foi parada
- NÃO está sendo cobrada por computação
- MAS está sendo cobrada pelo armazenamento EBS
- Pode ser reiniciada mantendo dados

### Terminating
- Processo de exclusão em andamento
- Não pode ser revertido
- Estado transitório

### Terminated
- Instância foi deletada permanentemente
- Nenhuma cobrança
- Dados foram perdidos (a menos que estejam em snapshot)

## Tipos de Instâncias

### T (Burstable Performance)
```
Uso: Desenvolvimento, teste, aplicações leves
Características:
  - CPU creditável
  - Performance baseline baixa
  - Performance burst quando necessário
Exemplos: t2.micro, t3.small, t4g.medium
Custo: Mais econômico
```

### M (General Purpose)
```
Uso: Aplicações equilibradas em produção
Características:
  - Balanço entre CPU, memória, rede
  - Performance consistente
  - Sem bursts
Exemplos: m5.large, m6i.xlarge, m7g.2xlarge
Custo: Moderado
```

### C (Compute Optimized)
```
Uso: Processamento intensivo, análise de dados
Características:
  - CPU de alta performance
  - Memória moderada
  - Ótima para cálculos complexos
Exemplos: c5.large, c6i.4xlarge, c7g.12xlarge
Custo: Mais caro
```

### R (Memory Optimized)
```
Uso: Bancos de dados, cache em memória, análise
Características:
  - RAM muito elevada
  - Ótima para dados em memória
  - Ideal para SAP, Oracle
Exemplos: r5.large, r6i.4xlarge, r7g.16xlarge
Custo: Caro
```

### I (Storage Optimized)
```
Uso: I/O intensivo, data warehousing
Características:
  - Armazenamento SSD rápido
  - Alto throughput I/O
  - Ideal para NoSQL, data lakes
Exemplos: i3.large, i3en.6xlarge
Custo: Caro
```

### G (Graphics - GPU)
```
Uso: Machine Learning, renderização 3D
Características:
  - GPU dedicada
  - Processamento paralelo
  - CUDA/OpenCL suportado
Exemplos: g4dn.xlarge, g5.2xlarge
Custo: Muito caro
```

## Selecionando o Tipo Certo

### Perguntas para Considerar:

1. **Qual é a carga típica?**
   - Constante → Use M ou C
   - Intermitente → Use T

2. **Qual é o requisito de memória?**
   - Baixo (< 4GB) → Use T
   - Moderado (4-32GB) → Use M
   - Alto (> 32GB) → Use R

3. **Qual é o requisito de CPU?**
   - Baixo (< 1 CPU) → Use T
   - Moderado (1-4 CPUs) → Use M
   - Alto (> 4 CPUs) → Use C

4. **Precisa de GPU?**
   - Sim → Use G ou P

5. **Qual é o padrão de acesso ao disco?**
   - Sequencial, alto throughput → Use I
   - Aleatório, latência baixa → Use EBS otimizado

## Comparação de Tamanhos

Dentro de cada tipo, existem tamanhos:

```
nano   < micro < small < medium < large < xlarge < 2xlarge < 4xlarge ...

t2.nano    (613 MB RAM, 1 CPU virtual)
t2.micro   (1 GB RAM, 1 CPU virtual)
t2.small   (2 GB RAM, 1 CPU virtual)
t2.medium  (4 GB RAM, 2 CPUs virtuais)
t2.large   (8 GB RAM, 2 CPUs virtuais)
t2.xlarge  (16 GB RAM, 4 CPUs virtuais)
```

## Pricing Models

### On-Demand
- Pague por hora ou segundo
- Sem compromisso
- Preço mais alto
- Ideal para: Testes, desenvolvimento

### Reserved Instances (RI)
- Desconto de até 72%
- Compromisso de 1 ou 3 anos
- Pode ser vendida no Marketplace
- Ideal para: Produção de longo prazo

### Spot Instances
- Desconto de até 90%
- Instância pode ser interrompida
- Preço flutua com demanda
- Ideal para: Batch jobs, análise de dados

### Dedicated Instances
- Hardware dedicado
- Sem compartilhamento com outros clientes
- Preço mais alto
- Ideal para: Conformidade regulatória

## AMI (Amazon Machine Image)

### O que é uma AMI?

Uma AMI é um template que contém:
- Sistema operacional (kernel, bootloader)
- Software pré-instalado
- Configurações de permissões
- Dados de aplicação

### Tipos de AMI

#### Quick Start (Oficial da AWS)
```
- Amazon Linux 2
- Ubuntu Server
- Red Hat Enterprise Linux
- SUSE Linux
- Windows Server
- macOS
```

#### Marketplace
```
- Imagens de terceiros
- Com software pré-configurado
- Podem ter custos adicionais
```

#### Community AMIs
```
- Criadas e compartilhadas por usuários
- Verificar segurança com cuidado
```

#### Custom AMIs
```
- Criadas pelo usuário
- Baseadas em instância existente
- Podem ser privadas ou compartilhadas
```

### Informações Importantes de uma AMI

- **AMI ID**: Identificador único (ami-xxxxxxxx)
- **Descrição**: O que contém
- **Arquitetura**: 32-bit, 64-bit, ARM
- **Root Device**: EBS ou Instance Store
- **Permissões**: Public, Private, Shared
- **Região**: Disponível em qual região

## Armazenamento em EC2

### EBS (Elastic Block Store)

Armazenamento persistente em nível de bloco.

```yaml
Características:
  - Persiste quando a instância é parada
  - Pode ser fotografado (snapshot)
  - Pode ser desanexado e reanexado
  - Replicado automaticamente na AZ

Tipos:
  gp3: Performance geral, boa relação preço/performance
  gp2: Performance geral herdado
  io1: IOPS de alta performance
  io2: IOPS ultra-alta performance
  st1: Throughput otimizado (big data)
  sc1: Cold storage (backup)
  standard: Herdado (não recomendado)
```

### Instance Store

Armazenamento efêmero (temporário) conectado fisicamente.

```yaml
Características:
  - Muito rápido
  - Gratuito com a instância
  - PERDIDO quando a instância é parada/terminada
  - Ideal para: Cache, dados temporários

Casos de uso:
  - Buffer temporário
  - Cache
  - Scratch space
  - Dados que podem ser recriados
```

## Redes

### VPC
- Rede privada virtual
- Isolamento total
- Subnets, route tables, NACLs
- NAT Gateways, Internet Gateways

### IP Público vs IP Privado
```
IP Privado:
  - Atribuído automaticamente
  - Não muda normalmente
  - Usa-se para comunicação interna
  - Formato: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16

Elastic IP:
  - IP público estático
  - Cobração se não associado
  - Útil para aplicações que precisam IP fixo
  - Pode ser reasssociado rapidamente
```

### Segurança em Rede
```
Security Groups:
  - Firewall stateful
  - Regras de ingress (entrada)
  - Regras de egress (saída)
  - Padrão: Nega tudo, libera o que você configura

NACLs:
  - Firewall stateless
  - Nível de subnet
  - Menos usado que Security Groups
```

---

**Próximo:** [Criar uma Instância EC2](02-criar-instancia.md)
