# 🚀 DIO - Gerenciamento de Instâncias EC2 na AWS

> Documentação e anotações consolidadas sobre gerenciamento de instâncias EC2 na Amazon Web Services (AWS)

## 📋 Índice

- [Sobre o Projeto](#sobre-o-projeto)
- [Objetivos de Aprendizagem](#objetivos-de-aprendizagem)
- [Conceitos Fundamentais](#conceitos-fundamentais)
- [Gerenciamento de Instâncias EC2](#gerenciamento-de-instâncias-ec2)
- [Boas Práticas](#boas-práticas)
- [Recursos Úteis](#recursos-úteis)

---

## 📝 Sobre o Projeto

Este repositório foi criado como parte do desafio da **Digital Innovation One (DIO)** com o objetivo de consolidar conhecimentos em gerenciamento de instâncias EC2 na AWS. O projeto documenta de forma clara e estruturada os conceitos aprendidos, servindo como material de referência para estudos e futuras implementações.

**Data de Criação:** 27 de maio de 2026

---

## 🎯 Objetivos de Aprendizagem

Ao concluir este desafio, você será capaz de:

- ✅ Aplicar conceitos de computação em nuvem em um ambiente prático
- ✅ Criar, configurar e gerenciar instâncias EC2 na AWS
- ✅ Documentar processos técnicos de forma clara e estruturada
- ✅ Utilizar o GitHub como ferramenta para compartilhamento de documentação técnica
- ✅ Implementar boas práticas de segurança e eficiência em ambientes cloud

---

## 📚 Conceitos Fundamentais

### O que é AWS EC2?

**Amazon Elastic Compute Cloud (EC2)** é um serviço de computação em nuvem que fornece capacidade de computação escalável sob demanda. É um dos serviços mais populares da AWS.

#### Características Principais:

| Característica | Descrição |
|---|---|
| **Elasticidade** | Aumente ou diminua a capacidade conforme necessário |
| **Controle Total** | Acesso root/administrativo às instâncias |
| **Flexibilidade** | Escolha entre diferentes tipos e tamanhos de instâncias |
| **Segurança** | Controle de rede, grupos de segurança e certificados |
| **Confiabilidade** | SLA de 99,99% de disponibilidade |
| **Custo-efetivo** | Pague apenas pelo que usar |

### Tipos de Instâncias EC2

```
Família de Instâncias:
├── T (General Purpose) - Aplicações leves, testes, desenvolvimento
├── M (General Purpose) - Aplicações equilibradas, produção
├── C (Compute Optimized) - Processamento intensivo, análise de dados
├── R (Memory Optimized) - Bancos de dados, cache, análise em memória
├── I (Storage Optimized) - I/O intensivo, data warehousing
├── G (GPU) - Machine Learning, renderização
└── H (High Memory) - Processamento de Big Data
```

### AMI (Amazon Machine Image)

Uma **AMI** é uma imagem pré-configurada que contém:
- Sistema operacional (Linux, Windows)
- Software pré-instalado
- Configurações de aplicação
- Permissões e perfis

**Tipos de AMI:**
1. **Amazon Linux** - Otimizada para AWS
2. **Ubuntu** - Distribuição Linux popular
3. **Windows Server** - Para aplicações Windows
4. **Custom** - Imagens personalizadas

---

## 🛠️ Gerenciamento de Instâncias EC2

### 1. Criar uma Instância EC2

#### Passo a Passo:

1. **Acessar o Console AWS**
   - Ir para AWS Management Console
   - Buscar por "EC2"
   - Clicar em "Instâncias"

2. **Clicar em "Executar instâncias"**
   - Escolher uma AMI
   - Selecionar tipo de instância
   - Configurar detalhes de rede
   - Adicionar armazenamento
   - Adicionar tags
   - Configurar grupo de segurança

3. **Revisar e Executar**
   - Verificar configurações
   - Gerar par de chaves (se novo)
   - Clicar em "Executar instâncias"

#### Exemplo de Configuração Recomendada:

```yaml
AMI: Ubuntu Server 22.04 LTS
Tipo de Instância: t3.micro (elegível para free tier)
Rede: VPC padrão
Sub-rede: Pública
IP Elástico: Sim
Grupos de Segurança:
  - SSH (22): Seu IP
  - HTTP (80): 0.0.0.0/0
  - HTTPS (443): 0.0.0.0/0
Armazenamento: 30 GB (gp3)
```

### 2. Conectar à Instância

#### Via SSH (Linux/Mac):

```bash
# Dar permissão ao arquivo de chave
chmod 400 sua-chave.pem

# Conectar à instância
ssh -i sua-chave.pem ec2-user@seu-ip-publico
```

#### Via RDP (Windows):

```
1. Baixar arquivo RDP
2. Obter senha usando a chave privada
3. Conectar usando Remote Desktop
```

### 3. Estados da Instância

```
pending (0-30 seg)
    ↓
running (operacional)
    ↓
stopping
    ↓
stopped (pausada, ainda incorre em custos de armazenamento)
    ↓
shutting-down
    ↓
terminated (encerrada, sem custos contínuos)
```

### 4. Operações Comuns

#### Parar Instância (retenha dados):
```
Console AWS → Clique com botão direito → Parar instância
```

#### Reiniciar Instância:
```
Console AWS → Clique com botão direito → Reiniciar instância
```

#### Terminar Instância (deletar permanentemente):
```
Console AWS → Clique com botão direito → Terminar instância
```

#### Usar Elastic IP (IP fixo):
```
Instâncias → Ações → Endereços Elásticos → Alocar endereço
→ Associar endereço elástico → Selecionar instância
```

---

## 💡 Boas Práticas

### Segurança

1. **Grupos de Segurança Restritivos**
   ```
   ❌ Evitar: 0.0.0.0/0 para SSH
   ✅ Usar: Seu IP específico ou VPN
   ```

2. **Gerenciamento de Chaves**
   - Guarde pares de chaves em local seguro
   - Use AWS Secrets Manager para credenciais
   - Nunca compartilhe chaves privadas

3. **IAM Roles**
   ```
   Use IAM Roles em vez de chaves de acesso nas instâncias
   ```

4. **Atualizações de Segurança**
   ```bash
   # Em instâncias Linux
   sudo apt-get update
   sudo apt-get upgrade
   ```

5. **VPC e Subnets**
   - Use subnets privadas para aplicações backend
   - Use NAT Gateway para saída de tráfego

### Performance

1. **Escolher Tipo Correto de Instância**
   - Analisar requisitos de CPU, memória, I/O
   - Começar com menor e escalar conforme necessário

2. **Monitoramento com CloudWatch**
   ```
   Métricas importantes:
   - CPU Utilization
   - Network In/Out
   - Disk Read/Write
   - Status Checks
   ```

3. **Auto Scaling**
   - Configurar grupos de auto scaling
   - Usar Load Balancers
   - Definir políticas baseadas em métricas

4. **Otimização de Armazenamento**
   ```
   gp3: Uso geral (recomendado)
   io1: Alta performance
   st1: Throughput otimizado
   sc1: Cold storage
   ```

### Custos

1. **Usar Free Tier**
   - t2.micro (750 horas/mês)
   - 30 GB de armazenamento EBS

2. **Reserved Instances**
   ```
   Economize até 72% em instâncias de longa duração
   ```

3. **Spot Instances**
   ```
   Até 90% de desconto para workloads interruptos
   ```

4. **Monitorar Custos**
   - Usar AWS Cost Explorer
   - Configurar billing alerts
   - Revisar relatórios mensais

### Backup e Recuperação

1. **Snapshots de EBS**
   ```
   Criar snapshots regulares para backup
   ```

2. **AMIs Customizadas**
   ```
   Salvar configurações como AMI para reutilização
   ```

3. **Disaster Recovery**
   ```
   - Multi-region deployment
   - Backup em S3
   - RTO/RPO definidos
   ```

---

## 📊 Monitoramento e Logs

### CloudWatch

```yaml
Métricas:
  CPU Utilization: % de CPU em uso
  Network In: Dados recebidos
  Network Out: Dados enviados
  Disk Operations: Leitura/escrita

Alarmes:
  Criar alertas para CPU > 80%
  Notificar via SNS
  Ação automática (scale up)
```

### EC2 Instance Status Checks

```
System Status Checks:
  ✅ Verifica hardware, rede, validação de dados
  
Instance Status Checks:
  ✅ Verifica SO, aplicação, kernel
  
Ação se falhar:
  → Parar e iniciar (novo hardware)
  → Contatar suporte AWS
```

---

## 🔗 Recursos Úteis

### Documentação Oficial

- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/)
- [AWS Best Practices](https://docs.aws.amazon.com/whitepapers/latest/aws-well-architected-framework/)

### Materiais DIO

- [Gerenciando EC2 instâncias da Amazon](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
- [GitHub Quick Start](https://github.com/skills/introduction-to-github)
- [GitBook - Formação GitHub Certification](https://gitbook.com)

### Ferramentas Recomendadas

- **AWS CLI**: Interface de linha de comando
- **Terraform**: Infrastructure as Code
- **AWS Systems Manager**: Gerenciamento de frota
- **CloudFormation**: Automação de recursos

---

## 📁 Estrutura do Repositório

```
dio-aws-ec2-management/
├── README.md                          # Este arquivo
├── docs/
│   ├── 01-conceitos-fundamentais.md   # Conceitos básicos de EC2
│   ├── 02-criar-instancia.md          # Passo a passo para criar instância
│   ├── 03-gerenciar-instancia.md      # Operações de gerenciamento
│   ├── 04-seguranca.md                # Práticas de segurança
│   ├── 05-monitoramento.md            # CloudWatch e métricas
│   ├── 06-custos.md                   # Otimização de custos
│   └── 07-troubleshooting.md          # Resolução de problemas
├── images/                            # Capturas de tela
│   ├── console-ec2.png
│   ├── criar-instancia.png
│   ├── grupos-seguranca.png
│   └── monitoramento.png
├── scripts/
│   ├── criar-instancia.sh             # Script de criação
│   ├── conectar-instancia.sh          # Script de conexão
│   └── configurar-instancia.sh        # Script de configuração
└── exemplos/
    ├── cloudformation.yaml             # Template CloudFormation
    ├── terraform.tf                    # Configuração Terraform
    └── user-data.sh                   # Script de inicialização
```

---

## 🎓 Próximos Passos

1. ✅ **Aprender EC2**: Consolidado
2. ✅ **Gerenciar Instâncias**: Em andamento
3. ⏭️ **Automatizar com Terraform**
4. ⏭️ **Usar CloudFormation**
5. ⏭️ **Implementar Auto Scaling**
6. ⏭️ **Load Balancing**

---

## 🤝 Contribuições

Este é um projeto de aprendizado pessoal, mas você pode:
- Sugerir melhorias
- Reportar erros
- Compartilhar experiências

---

## 📄 Licença

Este projeto está sob licença MIT. Veja [LICENSE](LICENSE) para mais detalhes.

---

## 👤 Autor

**Joana Iris**
- GitHub: [@joanairis](https://github.com/joanairis)
- Desafio DIO: Gerenciamento de EC2 na AWS

---

## 📞 Contato e Suporte

- 📧 Email: joanairis@example.com
- 🔗 LinkedIn: [seu-perfil]
- 💼 Portfólio: [seu-site]

---

**Última atualização:** 27 de maio de 2026

⭐ Se este repositório foi útil para você, considere deixar uma estrela!
