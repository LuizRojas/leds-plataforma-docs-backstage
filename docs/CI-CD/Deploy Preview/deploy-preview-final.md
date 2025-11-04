# Deploy Preview

Este documento explica **como funciona o Deploy Preview**, **para quê serve cada parte**, **o que roda onde**, e como tudo se conecta.

---

## O que é o Deploy Preview?

O Deploy Preview é um serviço que cria um ambiente **efêmero** para cada **Pull Request (PR)** na branch `develop`. Assim, os desenvolvedores podem ver e testar as mudanças **nos servidores**.

Quando o PR é fechado, o ambiente é destruído, **liberando os recursos** e removendo as portas usadas.

---

## Visão Geral

O fluxo tem **4 partes principais**:

1. **Dev** → Abre ou fecha PR.  
2. **Drone CI** → Publica a imagem docker, roda o script remoto e dispara o post comment.  
3. **VM do Deploy Preview** → Sobe containers isolados, controla portas e comenta no PR.  
4. **Preview Listener** → Observa fechamentos de PR e dispara limpeza.  

---

## **1 O Dev abre um Pull Request**

* Um **Pull Request (PR)** é aberto na branch `develop`.  
* O Drone é gatilhado pela abertura.

---

## **2 O Drone CI é acionado**

A pipeline do Deploy Preview no Drone tem 2 passos principais:

---

### **2.1 - Construir e publicar a imagem**

**O que faz:**

* Cria um `.env` temporário com variáveis de API e autenticação.  
* Builda a imagem com tag `pr-<PR_NUMBER>`.  
* Faz push para o **GitHub Container Registry (GHCR)**.  

Exemplo de tag gerada:

```
ghcr.io/leds-conectafapes/leds-conectafapes-frontend-portal-fapes:pr-${DRONE_PULL_REQUEST}
```

Secrets necessários nesta etapa:

| Secret                  | Descrição                                  |
| ----------------------- | ------------------------------------------ |
| `VITE_API_URL`          | URL da API                                 |
| `VITE_BASE_URL`         | URL base                                   |
| `VITE_BASE_URL_AUTH`    | URL base do Auth                           |
| `AUTHTOKEN_DEPLOY_PREVIEW` | Token de autenticação usado no preview |
| `GHCR_TOKEN`            | Token para push no GitHub Container Registry |
| `GITHUB_USERNAME`       | Usuário GitHub para login no GHCR          |

---

### **2.2 - Deploy remoto e comentário**

O segundo passo da pipeline usa o plugin `appleboy/drone-ssh` para acessar a máquina de preview.  
Lá ele roda dois scripts principais: `deploy-preview.py` e `post-comment.sh`.

#### **2.2.1 Executar Deploy**

```bash
python3 deploy-preview.py <REPO_NAME> <PR_NUMBER>
```

O script **`deploy-preview.py`** faz:  
1. Carrega `used_ports.txt`.  
2. Se já houver porta associada ao PR, reutiliza.  
3. Caso contrário, seleciona uma porta livre aleatória dentro do range e salva no arquivo.  
4. Cria o diretório do projeto: `./<REPO_NAME>/pr-<PR_NUMBER>`.  
5. Gera um `docker-compose.yml` a partir do `base-compose.yml`, substituindo variáveis `${REPO_NAME}`, `${PR_NUMBER}`, `${PORT_HTTP}`.  
6. Executa:  

```bash
docker compose down || true
docker compose pull
docker compose up -d
```

No final, imprime a porta alocada:  

```
[RESULT] PORT=<PORT_HTTP>
```

#### **2.2.2 Postar Comentário na PR**

Em seguida, o Drone executa:

```bash
./post-comment.sh ${DRONE_REPO} ${DRONE_PULL_REQUEST} $PORT $PREVIEW_APP_ID ${DRONE_REPO}
```

O script **`post-comment.sh`** faz:  
1. Monta um JWT (RS256) assinado com a chave privada do GitHub App.  
2. Obtém o `installation_id` via API.  
3. Troca JWT por Access Token.  
4. Valida se a PR existe.  
5. Busca comentários existentes.  
6. Se o comentário com o link ainda não existir, posta:  

```
Acesse aqui o deploy preview -> https://preview.conectafapes.leds.dev.br:<PORT>
```

Secrets necessários nesta etapa:

| Secret                   | Descrição                                  |
| ------------------------ | ------------------------------------------ |
| `PREVIEW_APP_ID`         | ID do GitHub App                           |
| `PREVIEW_APP_PRIVATE_KEY`| Chave privada usada para assinar JWT        |
| `SERVER_HOST_PREVIEW`    | Host da máquina de deploy preview           |
| `SERVER_USER`            | Usuário SSH                                |
| `SSH_KEY_PREVIEW`        | Chave SSH para acesso à VM                  |

---

## **3 O Preview Listener (Fechamento de PR)**

### O que é?

O **Preview Listener** é um servidor Flask que roda em container e escuta webhooks de **fechamento de PR** enviados pelo GitHub.

Quando detecta `pull_request.closed`, ele dispara a limpeza via SSH.

---

#### Como funciona?

1. Recebe `POST /webhook` do GitHub.  
2. Valida se o evento é `pull_request` com ação `closed`.  
3. Extrai `pr_number` e `repo_full_name` do payload.  
4. Executa via SSH:  

```bash
cleanup.sh <REPO_NAME> <PR_NUMBER>
```

---

#### **`cleanup.sh`**

O script de cleanup realiza:  
- Derrubar containers com `docker compose down`.  
- Remover container e volumes da PR.  
- Remover entrada correspondente no `used_ports.txt`.  
- Remover diretório do projeto (`/home/leds_admin/deploy-preview/<repo>/pr-<PR_NUMBER>`).  

Exemplo:

```bash
docker compose down
docker rm -f ${REPO_NAME}-pr-$PR_NUMBER || true
grep -v "^$REPO_NAME:$PR_NUMBER:" used_ports.txt > used_ports.tmp && mv used_ports.tmp used_ports.txt
rm -rf /home/leds_admin/deploy-preview/$REPO_NAME/pr-$PR_NUMBER
```

---

#### Secrets do Preview Listener

| Variável       | O que faz                                 |
| -------------- | ----------------------------------------- |
| `SSH_TARGET`   | IP/usuário alvo da máquina de preview     |
| `SCRIPT_PATH`  | Caminho do `cleanup.sh`                   |
| `SSH_KEY_PATH` | Caminho da chave SSH montada no container |

---

## **4 BPMN Completo**

O processo completo (abertura e fechamento de PR) pode ser visto no diagrama bpmn a seguir:

![alt text](diagrama.svg)
