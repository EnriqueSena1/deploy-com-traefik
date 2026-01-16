# 🚀 Deploy Automatizado com Traefik, Docker e GitHub Actions

Este projeto é um template de infraestrutura (IaC) que configura automaticamente um **Proxy Reverso com SSL (HTTPS)** usando Traefik e realiza o deploy contínuo de aplicações via GitHub Actions.

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Traefik](https://img.shields.io/badge/Traefik-24292e.svg?style=for-the-badge&logo=traefik&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/github%20actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white)

---

## 📂 Estrutura do Projeto

```text
.
├── .github/
│   └── workflows/
│       └── deploy.yml      # Pipeline de CI/CD
├── infra/                  # Configuração Global (Traefik)
│   ├── docker-compose.yml
│   └── traefik.yml
├── app-exemplo/            # Aplicação de Teste
│   └── docker-compose.yml
└── README.md

```

## 🛠️ Parte 1: Preparação do Servidor (VPS)
Acesse sua VPS via SSH como ROOT para preparar o ambiente inicial e criar o usuário de deploy seguro.

    1. Instalação de Dependências
        Rode este comando para garantir que Git e Docker estejam instalados:

        ```bash
        # Atualiza lista de pacotes
        sudo apt update && sudo apt upgrade -y

        # Instala Git se não existir
        command -v git >/dev/null 2>&1 || sudo apt install git -y

        # Instala Docker se não existir
        command -v docker >/dev/null 2>&1 || curl -fsSL [https://get.docker.com](https://get.docker.com) | sh

        # Cria a rede global do Traefik
        docker network ls | grep -q traefik-public || docker network create traefik-public
        ```
    2. Configuração do Usuário deployer
        Por segurança, não usamos o root para deploy. Criaremos um usuário específico:

        ```bash
        # Cria o usuário sem senha (acesso apenas via chave SSH)
        adduser --disabled-password --gecos "" deployer

        # Adiciona ao grupo do Docker
        usermod -aG docker deployer

        # Configura a pasta SSH
        mkdir -p /home/deployer/.ssh
        chmod 700 /home/deployer/.ssh
        touch /home/deployer/.ssh/authorized_keys
        chmod 600 /home/deployer/.ssh/authorized_keys
        chown -R deployer:deployer /home/deployer/.ssh
        ```
    3. Criando a Chave de Acesso (SSH Pipeline)
        Para que o GitHub Actions consiga entrar no seu VPS e fazer as atualizações sozinho, precisamos criar um par de chaves exclusivo para ele.

        No seu computador local, gere uma chave SSH sem senha:

        ```bash
        # O -f define um nome específico para não sobrescrever suas chaves pessoais
        ssh-keygen -t ed25519 -C "github-actions" -f deploy_key
        ```
        Quando pedir senha (passphrase), apenas aperte ENTER para deixar em branco.
        Isso vai criar dois arquivos na pasta onde você está:

        - `deploy_key`: (Sem extensão): É a Chave Privada (O segredo).
        - `deploy_key.pub`: É a Chave Pública (A fechadura).

        1. Instalar a "Fechadura" no VPS (Chave Pública)
            Copie o conteúdo da chave pública:

            ```bash
            cat deploy_key.pub
            # Copie o código que começa com "ssh-ed25519..."
            ```
            Acesse seu VPS e edite o arquivo de autorizações do usuário deployer:

            ```bash
            # No terminal do VPS
            sudo nano /home/deployer/.ssh/authorized_keys
            ```
            Cole o código na última linha do arquivo.
            Salve (Ctrl+O, Enter) e saia (Ctrl+X).

        2. Entregar a "Chave" para o GitHub (Chave Privada)
            Agora precisamos dar a chave para o robô do GitHub Actions.
            1. No seu computador, copie o conteúdo da chave privada:
   
            ```bash
            cat deploy_key
            # Copie TUDO, inclusive o "-----BEGIN OPENSSH PRIVATE KEY-----"
            ```
            2. Vá no seu repositório no GitHub:
            
                . Clique em Settings (Configurações).

                . No menu lateral esquerdo: Secrets and variables > Actions.

                . Clique no botão verde New repository secret.

            3. Preencha assim:
                . Name: SSH_KEY
                . Secret: (Cole o conteúdo da chave privada aqui).

            4. Clique no botão verde Add secret. 
   
## 🛠️ Parte 2: Conexão GitHub -> VPS (Repositório Privado)

Para que a VPS consiga baixar este projeto (se for privado), precisamos de uma Deploy Key.

1. Acesse a VPS como deployer:
    ```bash
    su - deployer
    ```
2. Gere a chave SSH:
   ```bash
   ssh-keygen -t ed25519 -C "github-actions" -f deploy_key

3. Exiba a chave pública:
    ```bash
    cat deploy_key.pub
    ```
4. Vá no seu repositório no GitHub:
   . Vá em Settings > Deploy Keys > Add deploy key.

   . Cole o conteúdo e salve.

## 🛠️ Parte 3: Configuração do Pipeline (GitHub Actions) 
    O arquivo .github/workflows/deploy.yml já está configurado para fazer o deploy. Você só precisa configurar as Secrets no repositório.

    Vá em Settings > Secrets and variables > Actions > New repository secret e adicione:

    Nome da Secret	Valor
    SSH_HOST	    O endereço IP da sua VPS
    SSH_KEY	        A sua chave PRIVADA SSH (aquela que permite entrar como deployer)
    SSH_USER	deployer  

## 🚀 Como fazer o Deploy
1. Faça qualquer alteração no código.

2. Faca o Commit e Push para a branch main.

3. Vá na aba Actions do GitHub e veja a mágica acontecer.

4. O script irá:

5. Conectar via SSH na VPS.

6. Atualizar o repositório.

7. Subir/Atualizar o Traefik (infra).

8. Subir/Atualizar a Aplicação de exemplo.

9. O SSL será gerado automaticamente em segundos.
