# Template Padrão de .drone.yml para CI/CD

Este documento apresenta um template padrão para um arquivo `.drone.yml`, projetado para automatizar o processo de **Build**, **Publish** e **Deploy** de aplicações containerizadas.

O fluxo é acionado por eventos de `push` nas branches `develop` e `test`, e utiliza o Docker-in-Docker (dind) para construir e publicar imagens no GitHub Container Registry (ghcr.io).

## Como Usar

Para utilizar este template, siga os passos abaixo:

### 1. Preencha os Placeholders

Antes de usar, você **precisa** substituir todos os placeholders (valores entre `<...>` ou com `..._DO_PROJETO`) no arquivo `.drone.yml` pelos valores corretos do seu projeto.

| Placeholder | Descrição | Exemplo |
| :--- | :--- | :--- |
| `<diretório_do_projeto>` | O caminho para a pasta que contém o `dockerfile` da sua aplicação. | `backend/app` |
| `<nome_org>` | O nome da sua organização ou usuário no GitHub. | `my-awesome-org` |
| `<nome_repo>` | O nome do repositório onde as imagens serão publicadas. | `my-project-api` |
| `DIGEST_<ID_UTILITÁRIO>` | Um identificador único para a variável de ambiente que armazena o digest da imagem. Use um nome que faça sentido para a imagem. | `DIGEST_API` |
| `<pasta_destino_deploy>` | A pasta dentro do repositório de deploy onde os manifestos estão. | `kubernetes/deployments` |
| `<arquivo_deploy.yaml>`| O nome do arquivo de manifesto (ex: Kubernetes) a ser atualizado. | `api-deployment.yaml` |

### 2. Configure os Secrets no Drone CI

Este pipeline precisa de credenciais para se autenticar no GitHub Container Registry. Você deve configurar os seguintes **secrets** nas configurações do seu repositório no Drone CI:

* **`GITHUB_USERNAME`**: Seu nome de usuário do GitHub.
* **`GHCR_TOKEN`**: Um Personal Access Token (PAT) do GitHub com permissões de `write:packages` para publicar as imagens.

## Estrutura do Pipeline

O arquivo é dividido em quatro pipelines que executam em sequência, dependendo da branch.

---

### 1. Pipeline: `Build`

Esta pipeline é a primeira a ser executada e tem a responsabilidade de construir as imagens Docker da aplicação para garantir que elas estão consistentes e sem erros.

* **Gatilho (Trigger):** Executada em `push` nas branches `develop` e `test`.
* **Função:** Navega até o diretório de cada projeto e executa o comando `docker build`. O template já vem preparado para construir mais de uma imagem, se necessário.

---

### 2. Pipeline: `Publish`

Após um build bem-sucedido, esta pipeline publica as imagens Docker no GitHub Container Registry (ghcr.io).

* **Gatilho (Trigger):** Executada em `push` nas branches `develop` e `test`.
* **Dependência:** Só executa após a pipeline `Build` ser concluída com sucesso (`depends_on: - Build`).
* **Função:**
    1.  Autentica-se no `ghcr.io` usando os secrets `GITHUB_USERNAME` e `GHCR_TOKEN`.
    2.  Constrói a imagem novamente, desta vez aplicando a tag com o nome da branch (ex: `ghcr.io/org/repo:develop`).
    3.  Envia a imagem tagueada para o `ghcr.io` com o comando `docker push`.

---

### 3. Pipeline: `Deploy`

Esta pipeline automatiza o deploy no ambiente de **desenvolvimento**.

* **Gatilho (Trigger):** Executada **apenas** em `push` na branch `develop`.
* **Dependência:** Só executa após a pipeline `Publish` ser concluída com sucesso (`depends_on: - Publish`).
* **Função:**
    1.  Instala as dependências necessárias (`git`, `wget`, `curl`, `yq`).
    2.  Autentica-se no `ghcr.io` e faz o `pull` da imagem com a tag `:develop`.
    3.  Clona o repositório que contém os arquivos de manifesto do deploy.
    4.  Usa o `docker inspect` para obter o **digest** da imagem (um identificador único e imutável), garantindo que a versão correta seja implantada.
    5.  Utiliza a ferramenta `yq` para atualizar o arquivo de manifesto (`.yaml`) com o digest da nova imagem.
    6.  Configura o `git`, adiciona as alterações, cria um commit e faz o `push` para o repositório de deploy, efetivando a atualização.

---

### 4. Pipeline: `Deploy-Test`

Esta pipeline é idêntica à de `Deploy`, mas focada no ambiente de **testes**.

* **Gatilho (Trigger):** Executada **apenas** em `push` na branch `test`.
* **Dependência:** Só executa após a pipeline `Publish` ser concluída com sucesso (`depends_on: - Publish`).
* **Função:** Realiza os mesmos passos da pipeline de `Deploy`, mas utilizando a imagem com a tag `:test` e atualizando o ambiente correspondente.

## Template do .drone.yml

``` yaml
kind: pipeline
type: docker
name: Build

trigger:
  event:
    - push
  branch:
    - develop
    - test

steps:
  - name: build
    image: docker:27.2.0-dind
    privileged: true
    volumes:
      - name: dockersock
        path: /var/run/docker.sock
    commands:
      - cd <diretório_do_projeto>
      - docker build -f dockerfile .
        #caso esteja a mexer com mais de uma imagem, seguir os próximos passos!
      - cd ../<diretório_do_projeto>
      - docker build -f dockerfile .
      - cd ..
      - echo "Build completed successfully"
services:
  - name: dind
    image: docker:27.1-dind
    privileged: true

volumes:
  - name: dockersock
    host:
      path: /var/run/docker.sock

---
kind: pipeline
type: docker
name: Publish

trigger:
  event:
    - push
  branch:
    - develop
    - test

depends_on:
  - Build

steps:
  - name: publish
    image: docker:27.2.0-dind
    privileged: true
    volumes:
      - name: dockersock
        path: /var/run/docker.sock
    environment:
      GITHUB_USERNAME:
        from_secret: GITHUB_USERNAME
      GITHUB_TOKEN:
        from_secret: GHCR_TOKEN
    commands:
      - echo $GITHUB_TOKEN | docker login ghcr.io -u $GITHUB_USERNAME --password-stdin
      - cd <diretório_do_projeto>
      - docker build -t ghcr.io/<nome_org>/<nome_repo>:${DRONE_COMMIT_BRANCH} -f dockerfile .
      - docker push ghcr.io/<nome_org>/<nome_repo>:${DRONE_COMMIT_BRANCH}
        #caso esteja a mexer com mais de uma imagem, seguir os próximos passos!
      - cd ../<diretorio_do_projeto>
      - docker build -t ghcr.io/<nome_org>/<nome_repo>:${DRONE_COMMIT_BRANCH} -f dockerfile .
      - docker push ghcr.io/<nome_org>/<nome_repo>:${DRONE_COMMIT_BRANCH}
      - cd .. 
      - echo "Publish completed successfully"

services:
  - name: dind
    image: docker:27.1-dind
    privileged: true

volumes:
  - name: dockersock
    host:
      path: /var/run/docker.sock

---
kind: pipeline
type: docker
name: Deploy

trigger:
  event:
    - push
  branch:
    - develop

depends_on:
  - Publish

volumes:
  - name: dockersock
    host:
      path: /var/run/docker.sock

steps:
  - name: deploy-stage
    image: docker:20.10.24-dind
    privileged: true
    environment:
      GITHUB_USERNAME:
        from_secret: GITHUB_USERNAME
      GITHUB_TOKEN:
        from_secret: GHCR_TOKEN
      DRONE_TAG: ${DRONE_COMMIT_BRANCH}-${DRONE_COMMIT_SHA}
    volumes:
      - name: dockersock
        path: /var/run/docker.sock
    commands:
      - apk add --no-cache git wget curl
      - wget [https://github.com/mikefarah/yq/releases/download/v4.44.1/yq_linux_amd64](https://github.com/mikefarah/yq/releases/download/v4.44.1/yq_linux_amd64) -O /usr/local/bin/yq
      - chmod +x /usr/local/bin/yq
      - echo "$GITHUB_TOKEN" | docker login ghcr.io -u "$GITHUB_USERNAME" --password-stdin
      - docker pull ghcr.io/<nome_org>/<nome_repo>:develop  
      - docker pull ghcr.io/<nome_org>/<nome_repo>:develop
      - git clone https://$GITHUB_USERNAME:$GITHUB_TOKEN@github.com/<nome_org>/<nome_repo>.git

      - DIGEST_<ID_UTILITÁRIO>=$(docker inspect --format='{{index .RepoDigests 0}}' ghcr.io/<nome_org>/<nome_repo>:develop | cut -d'@' -f2)
      - yq -i ".spec.template.spec.containers[0].image = \"ghcr.io/<nome_org>/<nome_repo>@$DIGEST_<ID_UTILITÁRIO>\"" <nome_repo>/<pasta_destino_deploy>/<arquivo_deploy.yaml>

      - DIGEST_<ID_UTILITÁRIO_2>=$(docker inspect --format='{{index .RepoDigests 0}}' ghcr.io/<nome_org>/<nome_repo>:develop | cut -d'@' -f2)
      - yq -i ".spec.template.spec.containers[1].image = \"ghcr.io/<nome_org>/<nome_repo>@$DIGEST_<ID_UTILITÁRIO_2>\"" <nome_repo>/<pasta_destino_deploy>/<arquivo_deploy.yaml>

      - cd <nome_repo>
      - echo "Deployment images updated successfully"
      - git config user.name "drone-ci"
      - git config user.email "drone@ci.leds"
      - git add <pasta_destino_deploy>/<arquivo_deploy.yaml>
      - git add <pasta_destino_deploy>/<arquivo_deploy.yaml>
      - |
        if ! git diff --cached --quiet; then
          git commit -m "[DEPLOY] Atualizando imagens para ${DRONE_TAG}"
        else
          echo "Nenhuma alteração para commitar"
        fi

      - git push
when:
     branch:
       include:
         - develop

---
kind: pipeline
type: docker
name: Deploy-Test

trigger:
  event:
    - push
  branch:
    - test

depends_on:
  - Publish

volumes:
  - name: dockersock
    host:
      path: /var/run/docker.sock

steps:
  - name: deploy-stage
    image: docker:20.10.24-dind
    privileged: true
    environment:
      GITHUB_USERNAME:
        from_secret: GITHUB_USERNAME
      GITHUB_TOKEN:
        from_secret: GHCR_TOKEN
      DRONE_TAG: ${DRONE_COMMIT_BRANCH}-${DRONE_COMMIT_SHA}
    volumes:
      - name: dockersock
        path: /var/run/docker.sock
    commands:
      - apk add --no-cache git wget curl
      - wget [https://github.com/mikefarah/yq/releases/download/v4.44.1/yq_linux_amd64](https://github.com/mikefarah/yq/releases/download/v4.44.1/yq_linux_amd64) -O /usr/local/bin/yq
      - chmod +x /usr/local/bin/yq
      - echo "$GITHUB_TOKEN" | docker login ghcr.io -u "$GITHUB_USERNAME" --password-stdin
      - docker pull ghcr.io/<nome_org>/<nome_repo>:test  
      - docker pull ghcr.io/<nome_org>/<nome_repo>:test
      - git clone https://$GITHUB_USERNAME:$GITHUB_TOKEN@github.com/<nome_org>/<nome_repo>.git

      - DIGEST_<ID_UTILITÁRIO>=$(docker inspect --format='{{index .RepoDigests 0}}' ghcr.io/<nome_org>/<nome_repo>:test | cut -d'@' -f2)
      - yq -i ".spec.template.spec.containers[0].image = \"ghcr.io/<nome_org>/<nome_repo>@$DIGEST_<ID_UTILITÁRIO>\"" <nome_repo>/<pasta_destino_deploy>/<arquivo_deploy.yaml>

      - DIGEST_<ID_UTILITÁRIO_2>=$(docker inspect --format='{{index .RepoDigests 0}}' ghcr.io/<nome_org>/<nome_repo>:test | cut -d'@' -f2)
      - yq -i ".spec.template.spec.containers[1].image = \"ghcr.io/<nome_org>/<nome_repo>@$DIGEST_<ID_UTILITÁRIO_2>\"" <nome_repo>/<pasta_destino_deploy>/<arquivo_deploy.yaml>

      - cd <nome_repo>
      - echo "Deployment images updated successfully"
      - git config user.name "drone-ci"
      - git config user.email "drone@ci.leds"
      - git add <pasta_destino_deploy>/<arquivo_deploy.yaml>
      - git add <pasta_destino_deploy>/<arquivo_deploy.yaml>
      - |
        if ! git diff --cached --quiet; then
          git commit -m "[DEPLOY] Atualizando imagens para ${DRONE_TAG}"
        else
          echo "Nenhuma alteração para commitar"
        fi

      - git push
when:
     branch:
       include:
         - test
```