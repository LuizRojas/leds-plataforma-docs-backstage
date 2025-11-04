# **Instalação do MkDocs**

## **Requisitos**

O MkDocs requer uma versão recente do **Python** e do **pip**,
instalados no seu sistema. Você pode verificar se já os possui
instalados usando o terminal:

```
python \--version
```
Python 3.8.2
```
pip \--version
pip 20.0.2 from /usr/local/lib/python3.8/site-packages (python 3.8)
```
Se já tiver essas ferramentas, pode pular diretamente para a seção **"Instalando o MkDocs"**. ([*MkDocs*](https://www.mkdocs.org/user-guide/installation/))

### **Instalando o Python**

Instale o Python usando o gerenciador de pacotes da sua preferência ou baixe-o diretamente do site oficial ([*python.org*](https://www.python.org/)) e execute o instalador adequado para seu sistema.

**Observação:** No Windows, ao instalar, certifique-se de marcar a opção
**"Add Python to PATH"**, pois muitas vezes vem desmarcada por padrão.
([*MkDocs*](https://www.mkdocs.org/user-guide/installation/))

### **Instalando o pip**

Se sua versão do Python já for recente, o `pip` provavelmente já está
instalado. Mesmo assim, pode ser uma boa ideia atualizá-lo para a versão
mais recente:

```
pip install \--upgrade pip
```
Se o pip não estiver instalado, você pode baixá-lo com o script
`get-pip.py` e instalá-lo assim:
```
python get-pip.py
```

### **Instalando o MkDocs**

Para instalar o MkDocs, execute o comando no terminal:
```bash
pip install mkdocs
```

Depois da instalação, verifique se o comando foi instalado corretamente:
```
mkdocs \--version
```

mkdocs, version 1.2.0 from /usr/local/lib/python3.8/site-packages/mkdocs
(Python 3.8)

***Pronto! Está instalado o teu MkDocs =]***


### Observação!
Caso esteja utilizando o Linux e venha a ter algum entrave, utilize os seguintes comandos:

```
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-pip -y
sudo apt install mkdocs -y
```