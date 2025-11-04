# Configuração de Checks com SonarQube na Pipeline
### 1. Acessar a interface do SonarQube
Abra o SonarQube disponível em:  
[http://sonarqube.conectafapes.leds.dev.br:9001/](http://sonarqube.conectafapes.leds.dev.br:9001/)

---

### 2. Criar um projeto no SonarQube
1. No menu, clique em **Projects** → **Create Project**.  
2. Escolha a opção **From Github.  
3. Selecione a organização correspondente, por exemplo:  `leds_conectafapes`
4. Selecione o repositório do LEDS correspondente, por exemplo: `leds-conectafapes-notifications`

---

### 3. Gerar o Token de Autenticação
1. Ainda na tela do projeto, selecione a opção **"Other CI"**.  
2. Clique em **Generate Token**, escolha o tipo do projeto.  
3. Copie o token gerado (ele só aparece uma vez).  
	1. Esse token será usado na pipeline.  

**Importante:** salve o token em um lugar seguro. Ele deve ser configurado como **secret** no Drone CI.

---

### 4. Configurar os Secrets no Drone CI
No repositório, configure os seguintes **secrets** no Drone CI:

- `SONAR_TOKEN_NOTIFICATION` → informe o token gerado no SonarQube.  
- `SONAR_ADDR` → endereço do SonarQube (No momento da criação dessa documentação essa é uma secret de organização no Drone CI):
	- [http://sonarqube.conectafapes.leds.dev.br:9001/](http://sonarqube.conectafapes.leds.dev.br:9001/)


---

### 5. Ajustar o `.drone.yml`
Adicione a seguinte etapa de **checks** no arquivo `.drone.yml` da pipeline:

```yaml
kind: pipeline
type: docker
name: Checks

steps:
- name: sonarqube-check
  image: mcr.microsoft.com/dotnet/sdk:8.0
  failure: ignore
  environment:
    sonarToken:
      from_secret: SONAR_TOKEN_NOTIFICATION
    sonarAddr:
      from_secret: SONAR_ADDR
  commands:
    - echo "Iniciando análise SonarQube"
    - cd src/<diretorio_da_solution>   # Ajuste para o diretório correto
    - dotnet tool install --global dotnet-sonarscanner
    - export PATH="$PATH:/root/.dotnet/tools"
    - dotnet sonarscanner begin /k:"<project_key>" /d:sonar.host.url="$sonarAddr" /d:sonar.token="$sonarToken" /d:sonar.qualitygate.wait=true # essa linha é gerada automaticamente no momento da configuração do token
    - dotnet build <solution>.sln # Ajuste para o tipo de build adequado
    - dotnet sonarscanner end /d:sonar.token="$sonarToken"
depends_on:
- Build

trigger:
branch:
  - main
  - test
  - develop
event:
  - pull_request
  - push
```
## Onde encontrar o `<project_key>`?

O `project_key` é exibido no **SonarQube** assim que você cria o projeto.  
Mas é possível consulta-lo em **Projects**→ **Projeto**→  **Project information**→ **Project Key**

Esse valor deve substituir o campo:

`/k:"<project_key>"`

---

## Resultado Esperado

Com a configuração acima:

- A cada **Pull Request** ou **Push** para `main`, `test` ou `develop`, o **SonarQube** será executado.
    
- O **Quality Gate** será avaliado, garantindo que o PR só avance se passar nas verificações.