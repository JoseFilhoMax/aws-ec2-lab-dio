# ☁️ Gerenciamento de Instâncias EC2 na AWS

> Desafio prático da plataforma DIO — documentação técnica sobre criação e gerenciamento de instâncias Amazon EC2.

---

## 📋 Índice

- [Sobre o Projeto](#sobre-o-projeto)
- [Pré-requisitos](#pré-requisitos)
- [Conceitos Fundamentais](#conceitos-fundamentais)
- [Passo a Passo: Criando uma Instância EC2](#passo-a-passo-criando-uma-instância-ec2)
- [Gerenciando a Instância](#gerenciando-a-instância)
- [Conectando via SSH](#conectando-via-ssh)
- [Boas Práticas e Segurança](#boas-práticas-e-segurança)
- [Limpeza de Recursos (Evitando Cobranças)](#limpeza-de-recursos)
- [Aprendizados e Conclusão](#aprendizados-e-conclusão)
- [Referências](#referências)

---

## Sobre o Projeto

Este repositório documenta a experiência prática de criação e gerenciamento de instâncias **Amazon EC2 (Elastic Compute Cloud)** como parte do desafio da [Digital Innovation One (DIO)](https://www.dio.me/).

O Amazon EC2 é um dos serviços mais fundamentais da AWS, permitindo provisionar servidores virtuais na nuvem com total controle sobre sistema operacional, recursos computacionais e configurações de rede — tudo isso sob demanda e com pagamento apenas pelo uso efetivo.

---

## Pré-requisitos

- Conta ativa na AWS (o Free Tier já é suficiente para este desafio)
- Acesso ao [AWS Management Console](https://console.aws.amazon.com/)
- Conhecimento básico de linha de comando (Linux/macOS) ou uso de SSH client no Windows (ex: PuTTY, MobaXterm, Windows Terminal)

---

## Conceitos Fundamentais

Antes de criar a primeira instância, é importante compreender os conceitos centrais do EC2:

### AMI — Amazon Machine Image
É o template que define o sistema operacional e as configurações iniciais da instância. Exemplos comuns: Amazon Linux 2023, Ubuntu Server 24.04, Windows Server 2022.

### Instance Type
Define a capacidade computacional (vCPUs, memória RAM, armazenamento de instância). A família `t2.micro` é elegível ao Free Tier e suficiente para testes e aprendizado.

| Família | Uso Ideal |
|--------|-----------|
| `t` (ex: t3.micro) | Uso geral, workloads variáveis — ideal para dev/test |
| `m` (ex: m6i.large) | Uso geral com desempenho balanceado |
| `c` (ex: c6g.xlarge) | Workloads intensivas em CPU |
| `r` (ex: r6i.2xlarge) | Workloads intensivas em memória |

### Security Group
Funciona como um firewall virtual que controla o tráfego de entrada (inbound) e saída (outbound) da instância. Cada regra especifica protocolo, porta e origem/destino permitidos.

### Key Pair
Par de chaves criptográficas (pública + privada) usado para autenticação SSH. A chave pública é associada à instância; a privada (`.pem`) fica com o usuário e **nunca deve ser compartilhada**.

### VPC e Subnet
A instância é criada dentro de uma **VPC (Virtual Private Cloud)**, que é uma rede virtual isolada. Dentro da VPC, as **subnets** dividem os espaços de endereçamento — públicas (acesso à internet) ou privadas.

### EBS — Elastic Block Store
Armazenamento em bloco persistente associado à instância. O volume root é criado automaticamente, e volumes adicionais podem ser anexados conforme necessidade.

---

## Passo a Passo: Criando uma Instância EC2

### 1. Acessar o serviço EC2

1. Faça login no [AWS Management Console](https://console.aws.amazon.com/)
2. Na barra de pesquisa, digite **EC2** e selecione o serviço
3. No painel do EC2, clique em **"Launch Instance"**

### 2. Configurar nome e AMI

- **Name**: Dê um nome descritivo à instância (ex: `dio-desafio-ec2`)
- **AMI**: Selecione **Amazon Linux 2023 AMI** (elegível ao Free Tier)

### 3. Selecionar o tipo de instância

- Selecione **`t2.micro`** (1 vCPU, 1 GB RAM — Free Tier elegível)

### 4. Criar ou selecionar Key Pair

- Clique em **"Create new key pair"**
- Nome: `dio-key-pair`
- Tipo: `RSA`
- Formato: `.pem` (Linux/macOS) ou `.ppk` (Windows + PuTTY)
- Clique em **"Create key pair"** — o arquivo será baixado automaticamente
- ⚠️ **Guarde este arquivo em local seguro. Não é possível recuperá-lo depois.**

### 5. Configurar Network Settings

- Mantenha a VPC padrão
- Marque **"Allow SSH traffic from"** → selecione `My IP` (mais seguro que `0.0.0.0/0`)
- Para servidores web: marque também as opções de HTTP (porta 80) e HTTPS (porta 443)

### 6. Configurar o armazenamento

- O padrão de **8 GiB gp3** já é suficiente para o desafio
- Free Tier oferece até 30 GB de armazenamento EBS

### 7. Revisar e lançar

- Revise o resumo no painel direito
- Clique em **"Launch Instance"**
- Aguarde o status mudar para `Running` (geralmente menos de 60 segundos)

---

## Gerenciando a Instância

### Ações disponíveis no console

Selecione a instância → menu **"Instance State"**:

| Ação | Descrição |
|------|-----------|
| **Start** | Liga uma instância parada |
| **Stop** | Para a instância (dados no EBS são preservados; não gera custo de compute) |
| **Reboot** | Reinicia sem alterar o estado |
| **Terminate** | **Destrói** a instância permanentemente (cuidado: dados no armazenamento de instância são perdidos) |
| **Hibernate** | Salva o estado da memória RAM no EBS antes de parar |

### Monitoramento básico

- Aba **"Monitoring"**: métricas de CPU, rede e disco via CloudWatch
- Aba **"Status Checks"**: verifica a saúde do hardware e do sistema operacional

---

## Conectando via SSH

### Linux / macOS

```bash
# Ajustar permissões da chave (obrigatório)
chmod 400 ~/Downloads/dio-key-pair.pem

# Conectar à instância
ssh -i ~/Downloads/dio-key-pair.pem ec2-user@<PUBLIC_IP_OR_DNS>
```

> Substitua `<PUBLIC_IP_OR_DNS>` pelo endereço público exibido no console EC2.  
> Para Ubuntu, o usuário padrão é `ubuntu`; para Amazon Linux, é `ec2-user`.

### Windows (via Windows Terminal / PowerShell)

```powershell
ssh -i C:\Users\SeuUsuario\Downloads\dio-key-pair.pem ec2-user@<PUBLIC_IP>
```

### Verificando a conexão

Após conectar, você verá o prompt do servidor remoto:

```
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|       
  ~~       \#/ ___   
   ~~       V~' '->  
    ~~~         /    
      ~~._.   _/     
         _/ _/       
       _/m/'           

[ec2-user@ip-172-31-xx-xx ~]$
```

---

## Boas Práticas e Segurança

- **Nunca expor a porta 22 (SSH) para `0.0.0.0/0`** em ambientes produtivos. Use `My IP` ou um bastion host.
- **Usar IAM Roles** para conceder permissões às instâncias, em vez de armazenar credenciais diretamente.
- **Ativar o AWS CloudTrail** para auditoria de todas as ações realizadas na conta.
- **Criar snapshots regulares** dos volumes EBS antes de alterações críticas.
- **Utilizar tags** consistentes para identificação e gestão de custos (ex: `Env=Dev`, `Project=DIO`, `Owner=Jose`).
- **Monitorar o Free Tier** via AWS Budgets para evitar surpresas na fatura.

---

## Limpeza de Recursos

> ⚠️ **Importante**: Para evitar cobranças indevidas ao fim do desafio, encerre os recursos criados.

```
1. EC2 Console → selecionar a instância
2. Instance State → Terminate Instance
3. Confirmar a terminação
4. Verificar também: Elastic IPs não associados, Snapshots e Volumes EBS órfãos
```

Um Elastic IP não associado a uma instância em execução **gera cobrança**. Sempre libere IPs elásticos não utilizados.

---

## Aprendizados e Conclusão

Com este desafio, foi possível compreender na prática:

- Como o Amazon EC2 abstrai a infraestrutura física, permitindo provisionar servidores em segundos
- A importância do modelo de segurança em camadas (Security Groups, Key Pairs, IAM)
- A diferença entre instâncias paradas (stop) e terminadas (terminate) e seus impactos no armazenamento e faturamento
- Como o modelo de precificação pay-as-you-go do EC2 se aplica na prática, com atenção especial ao Free Tier

O EC2 é a base sobre a qual grande parte das arquiteturas AWS é construída, e dominar seu gerenciamento é um passo fundamental para qualquer profissional de Cloud e DevOps.

---

## Referências

- [Documentação oficial — Gerenciando instâncias EC2 (AWS)](https://docs.aws.amazon.com/pt_br/toolkit-for-visual-studio/latest/user-guide/tkv-ec2-ami.html)
- [Amazon EC2 — Getting Started](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
- [AWS Free Tier — O que está incluído](https://aws.amazon.com/free/)
- [GitHub Quick Start — DIO](https://github.com/digitalinnovationone/github-quickstart)
- [Formação GitHub Certification — GitBook](https://aline-antunes.gitbook.io/formacao-fundamentos-github)

---

*Documentação produzida como entrega do desafio prático da [Digital Innovation One](https://www.dio.me/).*
