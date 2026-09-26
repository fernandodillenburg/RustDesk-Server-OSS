# 👨‍💻 Sobre o projeto

Este projeto faz parte do meu laboratório prático de Cloud, Linux, Docker, Networking e DevOps, com foco na construção e operação de infraestrutura self-hosted.

A proposta não é apenas instalar uma aplicação, mas documentar todo o ciclo:

Cloud ➔ Networking ➔ Linux ➔ Docker ➔ Security ➔ Deployment

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
                │    │   Rendezvous     │
                │    │                  │
                │    └── hbbr           │
                │        Relay          │
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

Canonical Ubuntu 20.04

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
Canonical Ubuntu 20.04

Networking:
VCN existente ou nova VCN

Public IPv4:
Enabled

Após a criação, obtenha o IP público:

PUBLIC_IP

<img width="1920" height="1545" alt="screencapture-cloud-oracle-srv" src="https://github.com/user-attachments/assets/6b6d0545-4875-47fe-b243-8d586185cb90" />


---

🔑 2. Acesso SSH

Acesse a VM com arquivo SSH gerado ou fornecido na criação da VM.

ssh ubuntu@<PUBLIC_IP>

Exemplo:

ssh ubuntu@203.0.113.10


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

<img width="1815" height="1029" alt="Screenshot_3" src="https://github.com/user-attachments/assets/c40ad743-384a-43d9-b005-a50b56ff0087" />


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

<img width="1920" height="1545" alt="screencapture-cloud-oracle-srv" src="https://github.com/user-attachments/assets/8a3607d3-3bbb-45d4-b0a6-8a979401e678" />

---

🔥 11. Firewall UFW

Verifique:

sudo ufw status

Ative o firewall:

sudo ufw enable

Antes disso, certifique-se de liberar SSH:

sudo ufw allow OpenSSH

Depois, libere somente as portas necessárias ao RustDesk.


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


ID Server:
<PUBLIC_IP>

Relay Server:
<PUBLIC_IP>

Key:
<CHAVE_PUBLICA>

<img width="1273" height="720" alt="Screenshot_1" src="https://github.com/user-attachments/assets/25525109-037c-4215-869c-f2abec9792b4" />

<img width="1919" height="1000" alt="Screenshot_2" src="https://github.com/user-attachments/assets/791f27ce-a0d0-408f-a669-bae3fa07e749" />



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
