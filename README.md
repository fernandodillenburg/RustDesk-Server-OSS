# 👨‍💻 Sobre o projeto

Este projeto faz parte do meu laboratório prático de Cloud, Linux, Docker, Networking e DevOps, com foco na construção e operação de infraestrutura self-hosted.

A proposta não é apenas instalar uma aplicação, mas documentar todo o ciclo:

Cloud
  ↓
Networking
  ↓
Linux
  ↓
Docker
  ↓
Security
  ↓
Deployment
  ↓
Monitoring
  ↓
Backup
  ↓
Troubleshooting

---
# 🚀 RustDesk Server OSS no Oracle Cloud (Always Free)

![RustDesk](https://img.shields.io/badge/RustDesk-OSS-orange?style=for-the-badge&logo=rustdesk&logoColor=white)
![Oracle Cloud](https://img.shields.io/badge/Oracle_Cloud-Always_Free-red?style=for-the-badge&logo=oracle&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Containerization-blue?style=for-the-badge&logo=docker&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Ubuntu-yellow?style=for-the-badge&logo=ubuntu&logoColor=white)


###
🖥️ RustDesk Server OSS — Self-Hosted na Oracle Cloud

Deploy de um servidor RustDesk Server OSS auto-hospedado em uma VM da Oracle Cloud Infrastructure (OCI), utilizando Docker Compose, configuração de rede, firewall e boas práticas de segurança e operação.

---

📌 Sobre o projeto

Este projeto documenta a implementação de uma infraestrutura self-hosted do RustDesk Server OSS em uma máquina virtual hospedada na Oracle Cloud Infrastructure.

O objetivo é disponibilizar uma infraestrutura própria para gerenciamento das conexões do RustDesk, evitando dependência de servidores públicos de relay/rendezvous e permitindo maior controle sobre:

- 🔐 Infraestrutura
- 🌐 Rede
- 🖥️ Clientes
- 📦 Containers
- 🔑 Chaves de acesso
- 📊 Operação
- 💾 Backup
- 🛡️ Segurança

Além de funcionar como um ambiente real de utilização, este projeto foi estruturado como um laboratório prático de DevOps / Cloud / Linux, contemplando infraestrutura, containers, networking e troubleshooting.

---

🏗️ Arquitetura

                         Internet
                            │
                            ▼
                 ┌─────────────────────┐
                 │    Oracle Cloud     │
                 │         OCI         │
                 └──────────┬──────────┘
                            │
                            ▼
                ┌───────────────────────┐
                │    OCI VCN / Subnet   │
                │                       │
                │  Security List / NSG  │
                └──────────┬────────────┘
                           │
                           ▼
                ┌───────────────────────┐
                │      VM Linux         │
                │                       │
                │  Docker               │
                │    │                  │
                │    ├── hbbs           │
                │    │   Rendezvous      │
                │    │                  │
                │    └── hbbr           │
                │        Relay           │
                │                       │
                └──────────┬────────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
        RustDesk Client            RustDesk Client

Componentes

Componente| Função
Oracle Cloud OCI| Provedor de infraestrutura
VM Linux| Host do servidor RustDesk
Docker| Runtime dos containers
Docker Compose| Orquestração
"hbbs"| Rendezvous / ID Server
"hbbr"| Relay Server
VCN| Rede virtual da OCI
Security List / NSG| Controle de tráfego da OCI
UFW| Firewall local da VM

---

🎯 Objetivos técnicos

Este laboratório demonstra conhecimentos em:

- Cloud Computing
- Oracle Cloud Infrastructure
- Linux
- Docker
- Docker Compose
- Networking
- TCP/UDP
- Firewall
- DNS
- SSH
- Infrastructure Operations
- Containerization
- Troubleshooting
- Hardening
- Documentação técnica

---

📋 Pré-requisitos

Oracle Cloud

É necessário possuir uma conta na:

Oracle Cloud Infrastructure

https://www.oracle.com/cloud/

Você precisará de uma VM com:

- IP público
- acesso SSH
- sistema Linux
- acesso administrativo ("sudo")
- regras de entrada configuráveis

A quantidade de CPU/RAM depende da quantidade de clientes esperados.

Para laboratório, uma VM pequena pode ser suficiente.

---

Conhecimentos recomendados

Antes de executar este tutorial, é recomendável possuir conhecimentos básicos de:

Linux
SSH
Docker
Networking
TCP / UDP
Firewall
DNS
Cloud

---

🖥️ Sistema operacional

Este tutorial utiliza como referência:

Ubuntu Server 24.04 LTS

Verifique o sistema:

cat /etc/os-release

Exemplo:

NAME="Ubuntu"
VERSION="24.04 LTS"

---

🌐 1. Criando a VM na Oracle Cloud

No Oracle Cloud Console:

Compute
  └── Instances
        └── Create Instance

Configure:

Name:
rustdesk-server

Image:
Ubuntu 24.04

Shape:
VM compatível com o workload desejado

Networking:
VCN existente ou nova VCN

Public IPv4:
Enabled

Após a criação, obtenha o IP público:

PUBLIC_IP

Exemplo:

203.0.113.10

«Nunca utilize o IP de exemplo acima em sua configuração.»

---

🔑 2. Acesso SSH

Acesse a VM:

ssh ubuntu@<PUBLIC_IP>

Exemplo:

ssh ubuntu@203.0.113.10

Verifique o usuário:

whoami

E o hostname:

hostnamectl

---

🔄 3. Atualização do sistema

Atualize os pacotes:

sudo apt update
sudo apt upgrade -y

Instale ferramentas básicas:

sudo apt install -y \
    curl \
    wget \
    git \
    ca-certificates \
    gnupg \
    lsb-release \
    ufw

---

🐳 4. Instalação do Docker

Remova versões antigas, caso existam:

sudo apt remove -y docker.io docker-doc docker-compose podman-docker containerd runc

Configure o repositório oficial do Docker:

sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg

Adicione o repositório:

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

Atualize o índice:

sudo apt update

Instale Docker Engine e Docker Compose:

sudo apt install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin

Valide:

docker --version

docker compose version

---

👤 5. Permitir Docker sem sudo

Adicione o usuário atual ao grupo Docker:

sudo usermod -aG docker $USER

Faça logout/login novamente:

exit

Reconecte:

ssh ubuntu@<PUBLIC_IP>

Teste:

docker ps

---

📁 6. Estrutura do projeto

Crie o diretório:

sudo mkdir -p /opt/rustdesk

Altere a propriedade:

sudo chown -R $USER:$USER /opt/rustdesk

Entre no diretório:

cd /opt/rustdesk

Estrutura esperada:

/opt/rustdesk/
├── docker-compose.yml
├── data/
└── README.md

---

🐳 7. Docker Compose

Crie:

nano docker-compose.yml

Utilize a imagem oficial/documentada do RustDesk Server OSS e configure os serviços "hbbs" e "hbbr".

Exemplo:

services:

  hbbs:
    container_name: rustdesk-hbbs
    image: rustdesk/rustdesk-server:latest
    command: hbbs
    volumes:
      - ./data:/root
    network_mode: host
    restart: unless-stopped

  hbbr:
    container_name: rustdesk-hbbr
    image: rustdesk/rustdesk-server:latest
    command: hbbr
    volumes:
      - ./data:/root
    network_mode: host
    restart: unless-stopped

«Nota: valide a tag/imagem atualmente recomendada na documentação oficial do RustDesk antes de colocar o ambiente em produção. Evite fixar "latest" em ambientes críticos sem uma estratégia de atualização.»

---

🚀 8. Inicializando o RustDesk Server

Execute:

docker compose up -d

Verifique os containers:

docker compose ps

Resultado esperado:

NAME             STATUS
rustdesk-hbbs    Up
rustdesk-hbbr    Up

Verifique os logs:

docker compose logs -f

Ou individualmente:

docker logs rustdesk-hbbs

docker logs rustdesk-hbbr

---

🔐 9. Chave pública do RustDesk

O servidor gera uma chave utilizada pelos clientes RustDesk.

Verifique o diretório:

ls -lah data/

Normalmente será possível encontrar arquivos relacionados às chaves do servidor.

Por exemplo:

cat data/id_ed25519.pub

Guarde a chave pública.

Ela será utilizada pelos clientes para validar o servidor RustDesk.

«Nunca compartilhe a chave privada "id_ed25519".»

---

🌐 10. Configuração de firewall na Oracle Cloud

Esta é uma das etapas mais importantes.

A Oracle Cloud possui controles de rede externos à VM.

Dependendo da configuração utilizada, será necessário liberar as portas na:

VCN
 └── Subnet
      └── Security List / Network Security Group

As portas exatas devem ser conferidas na documentação oficial da versão do RustDesk utilizada.

Como referência, o servidor RustDesk utiliza portas para:

TCP
UDP
Rendezvous
Relay

Não abra indiscriminadamente todas as portas.

Utilize apenas as portas necessárias para os serviços efetivamente configurados.

---

🔥 11. Firewall UFW

Verifique:

sudo ufw status

Ative o firewall:

sudo ufw enable

Antes disso, certifique-se de liberar SSH:

sudo ufw allow OpenSSH

Depois, libere somente as portas necessárias ao RustDesk.

Exemplo:

sudo ufw allow <PORTA_TCP>/tcp
sudo ufw allow <PORTA_UDP>/udp

Verifique:

sudo ufw status verbose

⚠️ Importante

O tráfego precisa ser permitido em todas as camadas relevantes:

Internet
   ↓
Oracle VCN / NSG / Security List
   ↓
VM
   ↓
UFW
   ↓
Docker
   ↓
RustDesk

Liberar uma porta apenas no UFW não garante que ela estará acessível pela Internet.

---

🔎 12. Testando as portas

A partir de outra máquina:

nc -vz <PUBLIC_IP> <PORTA>

Exemplo:

nc -vz 203.0.113.10 21115

Para verificar portas TCP localmente:

sudo ss -lntp

Para UDP:

sudo ss -lnup

---

🖥️ 13. Configuração do cliente RustDesk

No cliente RustDesk, acesse as configurações de rede.

Configure o servidor de ID/Rendezvous e Relay conforme o domínio/IP e portas definidos na infraestrutura.

Exemplo conceitual:

ID Server:
rustdesk.example.com

Relay Server:
rustdesk.example.com

Key:
<CHAVE_PUBLICA>

«Use o domínio ou IP público real do seu servidor.»

---

🌍 14. DNS

Para uma implantação profissional, recomenda-se utilizar um domínio.

Exemplo:

rustdesk.example.com

Crie um registro:

A
rustdesk.example.com
→
PUBLIC_IP

Verifique:

dig rustdesk.example.com

ou:

nslookup rustdesk.example.com

---

🔒 15. Segurança

Uma implantação exposta à Internet deve ser tratada como infraestrutura de produção.

SSH

Evite autenticação por senha:

PasswordAuthentication no

Prefira:

SSH Key Authentication

Considere também:

- desabilitar login SSH direto de "root"
- restringir origem do SSH
- utilizar chaves SSH
- manter o sistema atualizado
- monitorar logs

---

Docker

Verifique containers:

docker ps

Imagens:

docker images

Logs:

docker compose logs

Evite executar containers privilegiados sem necessidade.

---

🔄 16. Atualização

Antes de atualizar:

docker compose pull

Revise as alterações.

Depois:

docker compose up -d

Valide:

docker compose ps

E:

docker compose logs --tail=100

Estratégia recomendada

Em produção:

Backup
   ↓
Pull da nova versão
   ↓
Atualização
   ↓
Health check
   ↓
Monitoramento
   ↓
Rollback se necessário

Evite atualizar diretamente em produção sem conhecer as alterações da versão.

---

💾 17. Backup

Os dados persistidos estão no diretório:

/opt/rustdesk/data

Faça backup:

tar -czvf rustdesk-backup-$(date +%F).tar.gz data/

Exemplo:

rustdesk-backup-2026-09-26.tar.gz

Para um ambiente real, o backup deve ser armazenado fora da própria VM.

Possibilidades:

Object Storage
Storage externo
Backup server
Outra região

---

📊 18. Monitoramento

Verifique recursos da VM:

htop

Memória:

free -h

Disco:

df -h

Containers:

docker stats

Logs:

docker compose logs --tail=100

Status:

docker compose ps

---

🧪 19. Checklist de validação

Após a instalação:

[ ] VM acessível via SSH
[ ] Sistema operacional atualizado
[ ] Docker instalado
[ ] Docker Compose funcionando
[ ] hbbs em execução
[ ] hbbr em execução
[ ] Chaves geradas
[ ] Portas configuradas na OCI
[ ] Firewall UFW configurado
[ ] DNS resolvendo
[ ] Cliente RustDesk configurado
[ ] Conexão entre clientes validada
[ ] Relay validado
[ ] Backup configurado
[ ] Logs verificados

---

🛠️ 20. Troubleshooting

Container não inicia

Verifique:

docker compose ps

Depois:

docker compose logs hbbs

e:

docker compose logs hbbr

---

Porta não acessível

Verifique em sequência:

1. RustDesk está executando?
2. A porta está sendo escutada?
3. UFW permite a porta?
4. OCI Security List permite?
5. OCI NSG permite?
6. Existe outro firewall?
7. DNS aponta para o IP correto?

Comandos:

sudo ss -lntup

sudo ufw status

docker compose ps

---

Cliente não encontra o servidor

Verifique:

DNS
IP público
Portas
Firewall
Key
Configuração do ID Server
Configuração do Relay Server

Teste DNS:

dig rustdesk.example.com

Teste conectividade:

nc -vz rustdesk.example.com <PORTA>

---

🧰 21. Comandos úteis

Iniciar

docker compose up -d

Parar

docker compose down

Reiniciar

docker compose restart

Atualizar

docker compose pull
docker compose up -d

Logs

docker compose logs -f

Status

docker compose ps

Recursos

docker stats

---

📁 Estrutura final

rustdesk-server/
│
├── docker-compose.yml
│
├── data/
│   ├── id_ed25519
│   ├── id_ed25519.pub
│   └── ...
│
└── README.md

«Arquivos contendo chaves privadas não devem ser commitados no Git.»

---

🚫 .gitignore

Crie:

nano .gitignore

Conteúdo:

# RustDesk persistent data
data/

# Private keys
*.key
*.pem
id_ed25519
id_ed25519.*

# Environment files
.env
.env.*

# Logs
*.log

# Backup files
*.tar
*.tar.gz
*.zip

# OS
.DS_Store
Thumbs.db

---

🧱 Melhorias futuras

Este laboratório pode evoluir para uma arquitetura mais próxima de um ambiente profissional.

Infraestrutura como Código

Implementar OCI com:

Terraform

Exemplo:

Terraform
   │
   ├── VCN
   ├── Subnet
   ├── Security List
   ├── NSG
   ├── Compute Instance
   └── Public IP

---

CI/CD

Adicionar GitHub Actions para:

Lint
   ↓
Validation
   ↓
Docker Compose validation
   ↓
Security scanning
   ↓
Deployment

---

Observabilidade

Adicionar:

Prometheus
Grafana
Loki
Alertmanager

Arquitetura:

RustDesk
   │
   ▼
Metrics / Logs
   │
   ├── Prometheus
   │
   ├── Loki
   │
   └── Grafana

---

Automação

Criar scripts para:

install.sh
backup.sh
restore.sh
healthcheck.sh
update.sh

---

🔐 Considerações de segurança

Este projeto é destinado a fins educacionais, laboratoriais e de infraestrutura própria.

Antes de utilizar em produção:

- valide as portas necessárias na versão instalada;
- utilize versões fixadas das imagens;
- implemente backup externo;
- proteja as chaves privadas;
- mantenha o sistema operacional atualizado;
- restrinja o acesso SSH;
- utilize DNS apropriado;
- monitore os serviços;
- documente procedimentos de recuperação;
- teste restauração de backup;
- revise periodicamente as regras de firewall.

---

📚 Referências

- RustDesk
  https://rustdesk.com/

- RustDesk Server OSS
  https://rustdesk.com/docs/en/self-host/

- RustDesk GitHub
  https://github.com/rustdesk/rustdesk-server

- Oracle Cloud Infrastructure
  https://docs.oracle.com/en-us/iaas/

- Docker Documentation
  https://docs.docker.com/

- Docker Compose
  https://docs.docker.com/compose/

---
Built with Linux + Docker + Oracle Cloud + RustDesk OSS.
