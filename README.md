# ☁️ AWS NETWORK TRAFFIC & SECURITY LAB
### Implementing and Controlling Network Traffic on AWS

![AWS](https://img.shields.io/badge/Cloud-AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Networking](https://img.shields.io/badge/Networking-VPC-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Security](https://img.shields.io/badge/Security-Network%20Controls-2E8B57?style=for-the-badge)
![Monitoring](https://img.shields.io/badge/Monitoring-CloudWatch-BC6C25?style=for-the-badge&logo=amazoncloudwatch&logoColor=white)
![Status](https://img.shields.io/badge/Status-Lab%20Completed-2E8B57?style=for-the-badge)

> **Laboratório prático de infraestrutura em nuvem, segmentação de rede, controle de tráfego e monitoramento na Amazon Web Services (AWS).**

Este projeto documenta minha experiência prática no laboratório **Implementing and Controlling Network Traffic on AWS**, no qual trabalhei com os principais mecanismos de segurança de rede da AWS e configurei o monitoramento do tráfego de uma infraestrutura que utiliza balanceador de carga, servidor web e banco de dados.

O objetivo deste repositório é registrar a arquitetura estudada, as configurações realizadas, os conceitos técnicos aplicados e as evidências visuais do laboratório.

---

## 📌 Informações do projeto

| Item | Detalhes |
|---|---|
| Projeto | Implementing and Controlling Network Traffic on AWS |
| Plataforma | AWS Academy / ambiente de laboratório |
| Laboratório | SPL-TF-200-SISBPN-1 |
| Versão | 1.0.9 |
| Conclusão | 27 de outubro de 2026 |
| Certificado | RDUYLCZ0I929 |
| Região AWS | US West (Oregon) — `us-west-2` |
| Área técnica | Cloud Networking, Network Security e Monitoring |
| Status | Laboratório concluído; documentação em evolução |

## 🎯 Objetivos técnicos

- Compreender a organização de uma rede virtual na AWS usando Amazon VPC.
- Trabalhar com sub-redes públicas e privadas.
- Configurar Security Groups para controlar conexões entre recursos.
- Implementar Network ACLs para filtrar tráfego no nível das sub-redes.
- Aplicar regras de entrada e saída com protocolos, portas e intervalos CIDR.
- Configurar VPC Flow Logs para registrar informações sobre o tráfego de rede.
- Integrar os registros ao Amazon CloudWatch Logs.
- Investigar a relação entre regras de segurança, conectividade e monitoramento.

## 🏗️ Arquitetura estudada

A infraestrutura do laboratório utiliza uma arquitetura de rede segmentada, com balanceador de carga, servidor web e banco de dados distribuídos em sub-redes distintas.

```text
                         CLIENTE
                            |
                            | HTTPS / TCP 443
                            v
                  +----------------------+
                  |   Network Load       |
                  |      Balancer        |
                  +----------------------+
                            |
                     Sub-redes públicas
                            |
                  +----------------------+
                  |    Servidor Web      |
                  |   Apache / WordPress |
                  +----------------------+
                            |
                       MySQL / TCP 3306
                            |
                  +----------------------+
                  |   Banco de Dados     |
                  |     Sub-rede privada |
                  +----------------------+

       CONTROLE DE REDE
       ├── Security Groups
       ├── Network ACLs
       └── VPC Flow Logs
                  |
                  v
          Amazon CloudWatch Logs
```

*Diagrama conceitual da arquitetura do laboratório. Não representa uma exportação automática dos recursos reais nem confirma que todos os testes de conectividade foram bem-sucedidos.*

### Segmentação de rede

A VPC utilizada no laboratório possui o bloco CIDR `10.10.0.0/16`.

| Sub-rede | CIDR | Finalidade |
|---|---|---|
| Load Balancer Subnet 1 | `10.10.1.0/24` | Balanceador de carga |
| Load Balancer Subnet 2 | `10.10.2.0/24` | Balanceador de carga |
| Web Server Subnet | `10.10.10.0/24` | Servidor web |
| Database Server Subnet | `10.10.20.0/24` | Banco de dados |

A segmentação permite organizar os recursos por função e aplicar controles de rede específicos para cada camada.

## 🔐 1. Security Groups

Os Security Groups funcionam como firewalls virtuais associados aos recursos da AWS. Suas regras controlam o tráfego permitido de entrada e saída.

### WebServerSG

Configurações trabalhadas:

- Permissão de entrada HTTPS pela porta TCP `443`.
- Permissão de saída MySQL pela porta TCP `3306` para o Security Group do banco de dados.

### DatabaseServerSG

Configurações trabalhadas:

- Permissão de entrada MySQL pela porta TCP `3306`, com origem no Security Group do servidor web.
- Revisão das regras de entrada e saída disponibilizadas pelo laboratório.

**Aprendizado:** utilizar referências a Security Groups permite definir relações de confiança entre camadas da aplicação sem depender exclusivamente de endereços IP individuais.

## 🧱 2. Network ACLs

As Network ACLs são controles de tráfego aplicados no nível das sub-redes. Diferentemente dos Security Groups, são stateless: as regras de entrada e saída precisam considerar os dois sentidos da comunicação.

Foram trabalhadas ACLs específicas para:

- Balanceador de carga.
- Servidor web.
- Banco de dados.

### Principais regras estudadas

| Tráfego | Porta / intervalo | Finalidade |
|---|---:|---|
| HTTPS | TCP 443 | Acesso seguro ao serviço web |
| MySQL | TCP 3306 | Comunicação entre servidor web e banco de dados |
| Portas efêmeras | TCP 1024–65535 | Tráfego de retorno e conexões de rede conforme o fluxo previsto |

As regras de cada ACL foram organizadas por número, tipo de tráfego, origem ou destino e ação de permitir ou negar.

**Aprendizado:** a ordem das regras, os intervalos de portas e os CIDRs influenciam diretamente o resultado do controle de tráfego.

## 📊 3. VPC Flow Logs e Amazon CloudWatch

Uma das etapas centrais foi configurar o registro do tráfego de rede da VPC.

### Configuração documentada

| Parâmetro | Valor |
|---|---|
| Nome do Flow Log | `vpc-flow-logs-para-blog` |
| Identificador | `fl-0bf165a0c8368a340` |
| Estado apresentado | Ativo |
| Destino | CloudWatch Logs |
| Grupo de logs | `registros-de-acesso-ao-blog` |
| Intervalo de agregação | 1 minuto |
| Filtro | Todo o tráfego |
| Função IAM selecionada | `VpcFlowLogsRole` |

O grupo de logs foi criado pelo fluxo de configuração do VPC Flow Logs, utilizando a função disponibilizada no laboratório.

### Campos do formato personalizado

O formato configurado foi destinado a registrar informações como:

- `account-id`
- `interface-id`
- `srcaddr`
- `srcport`
- `dstaddr`
- `dstport`
- `subnet-id`
- `flow-direction`
- `action`

Esses campos permitem relacionar interfaces, endereços IP, portas, sub-redes, direção do tráfego e ação registrada.

### Por que isso é importante?

O VPC Flow Logs permite investigar padrões de tráfego, identificar conexões aceitas ou rejeitadas e apoiar a análise de conectividade e segurança.

Os registros não substituem logs de aplicação, e sua presença não significa que toda conexão foi autorizada ou que um serviço está funcionando corretamente.

## 🔎 4. Investigação de tráfego permitido e rejeitado

O exercício propõe gerar acessos ao WordPress por HTTPS e tentar uma conexão HTTP pela porta 80, para depois investigar os registros no CloudWatch.

### Cenários previstos no laboratório

| Cenário | Resultado esperado |
|---|---|
| Tráfego HTTPS autorizado | Registro compatível com `ACCEPT` |
| Tráfego HTTP bloqueado pelas regras aplicáveis | Registro compatível com `REJECT` |
| Tráfego em portas efêmeras | Pode estar relacionado a conexões de retorno ou outros fluxos de rede |

**Nota sobre validação:** a configuração do Flow Log foi concluída e o recurso apareceu como ativo. Entretanto, a conectividade final do WordPress e a consulta dos eventos reais `ACCEPT` e `REJECT` não foram confirmadas nesta documentação. Os cenários acima descrevem o objetivo do exercício, não resultados de testes que não foram verificados.

## 🧠 5. Conhecimentos desenvolvidos

Durante o laboratório, trabalhei com os seguintes conceitos:

- Amazon VPC e endereçamento CIDR.
- Segmentação de rede por sub-redes.
- Security Groups e controle stateful de conexões.
- Network ACLs e filtragem stateless.
- Protocolos TCP, HTTPS e MySQL.
- Portas efêmeras e tráfego de retorno.
- VPC Flow Logs.
- Amazon CloudWatch Logs.
- Investigação de conectividade e diagnóstico de regras de rede.
- Relação entre segurança de rede e disponibilidade de aplicações.

## 🖼️ 6. Evidências do laboratório

Os prints abaixo devem ser adicionados ao repositório para documentar as etapas realmente executadas.

Sugestão de organização:

```text
aws-network-traffic-security-lab/
├── README.md
├── docs/
│   └── screenshots/
│       ├── 01-lab-overview.png
│       ├── 02-vpc-subnets.png
│       ├── 03-webserver-security-group.png
│       ├── 04-database-security-group.png
│       ├── 05-load-balancer-nacl.png
│       ├── 06-webserver-nacl.png
│       ├── 07-database-nacl.png
│       ├── 08-vpc-flow-log.png
│       ├── 09-cloudwatch-log-group.png
│       └── 10-lab-completion-certificate.png
└── LICENSE
```

Os nomes são sugestões: substitua-os pelos nomes reais dos seus arquivos e mantenha apenas as evidências que você possui.

### Como inserir as imagens no README

Depois de enviar os prints para `docs/screenshots/`, adicione referências como estas:

```markdown
### Configuração dos Security Groups

![Security Groups](docs/screenshots/03-webserver-security-group.png)

### Configuração do VPC Flow Logs

![VPC Flow Logs](docs/screenshots/08-vpc-flow-log.png)

### Conclusão do laboratório

![Certificado de conclusão](docs/screenshots/10-lab-completion-certificate.png)
```

Recomendo ocultar dados de conta, identificadores de sessão, informações sensíveis e outros detalhes que não sejam necessários para demonstrar o aprendizado.

## 🏆 7. Certificado de conclusão

**Laboratório:** Implementing and Controlling Network Traffic on AWS  
**Conclusão:** 27 de outubro de 2026  
**ID do certificado:** `RDUYLCZ0I929`

A conclusão registra a participação no laboratório prático e o estudo dos mecanismos de controle de tráfego de rede da AWS.

O certificado pode ser apresentado junto às evidências do projeto, respeitando as regras da plataforma sobre compartilhamento.

## 💼 8. Aplicação profissional

Os conceitos estudados têm relação com atividades de:

- Suporte técnico N2/N3.
- Redes e infraestrutura.
- Diagnóstico de conectividade.
- Administração de ambientes em nuvem.
- Segurança de rede.
- Monitoramento e análise de logs.
- Investigação de falhas de comunicação entre aplicações e bancos de dados.

A prática ajuda a compreender como regras de rede podem afetar a comunicação entre componentes de uma aplicação e como os registros de tráfego podem auxiliar no diagnóstico.

## 📚 9. Referências oficiais

- [Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
- [CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)

## 📈 Status do projeto

**LABORATÓRIO CONCLUÍDO — DOCUMENTAÇÃO EM EVOLUÇÃO**

Este repositório tem finalidade educacional e de portfólio. Documenta a configuração estudada, os conceitos trabalhados e as evidências disponíveis do laboratório. Não representa um ambiente de produção nem comprova experiência profissional administrando infraestrutura AWS em produção.

---

**Paulo Henrique Santana Motta**  
Portfólio técnico — Redes | Infraestrutura de TI | Cloud | Segurança de Rede
