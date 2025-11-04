# Open-VPN: Instalação no Linux

## Tutorial de instalação da Open-VPN no Linux

O cliente OpenVPN dá acesso à rede interna do LEDS. A rede interna
hospeda serviços que só podem ser acessados por meio da VPN.

Instale o OpenVPN com o comando:
```bash
sudo apt-get install openvpn
```

Como conseguir acesso?

-   Se já não possuir o bundle de acesso (geralmente um .zip do OpenVPN,
    juntamente com seu user e password), solicite-o ao SysAdmin.
-   Com o bundle em mãos, vá até a pasta em que foi baixado o arquivo
    .zip, podendo ser por meio do terminal ou pela GUI mesmo e descompacte
    os arquivos.

    ![](imagens/linux-vpn/100000010000037100000225BBADF52B.png)
    
    ***Após descompactar, deverá ter algo parecido com isso***.


** **    



-  Vá ao menu de opções do teu desktop/notebook.

![](imagens/linux-vpn/1000000100000195000001624439CDFE.png)

 ***Clique na seta em "Wired".***

---

![](imagens/linux-vpn/100000010000017F000000BBBC712D66.png) 


***Clique em "Wired Settings".***


---
-   Neste menu, em VPN tu irá clicar no ícone "+". (Obs: Caso esteja
    pelo "Wi-Fi", clique na opção "All Networks" e vá até "Network").
![](imagens/linux-vpn/10000001000002240000011E5804904D.png)
---

![](imagens/linux-vpn/10000001000003A9000002476AF30824.png)


***Clique em "Import from file...".***

---


-   Caso tenha sucesso, abrirá o menu de configuração da "VPN".

![](imagens/linux-vpn/10000001000002F50000021E05727FED.png)


***Na pasta desejada, selecione o arquivo ".ovpn", depois clique em
"Open".***

---
-   Em "Identity", irá de alterar apenas a opção "User private key".


![](imagens/linux-vpn/10000001000002F50000021EFE9C7447.png)

---

- Em "User key password", deixar a última opção marcada. 
![](imagens/linux-vpn/idpsw.png)
***Marcar "The password is not required"***

---

- Após isso, vá até "IPv4".

![](imagens/linux-vpn/100000010000019800000110C3073150.png)


***Marque a opção "Use this connection only..."***

-   Clique em aplicar e então terá sua "VPN" instalada e pronta para
    uso!
---

Para testar se o acesso foi validado e para trocar a sua senha de
usuário, é de **SUMA IMPORTÂNCIA** criar uma nova senha segura na
interface do firewall. Siga estes passos:

-   Acesse **fw.intranet.leds.dev.br**. Se tudo deu certo, você estará na interface. Se ocorreu algum erro de rede, muito provavelmente sua VPN foi configurada de forma errada. Solicite ajuda para o SysAdmin.
  
![](imagens/linux-vpn/10000001000001A600000148B3971CA4.png)

***Landing page do firewall.***

**    ** 

-   Insira seu user e senha. O user geralmente é nome.sobrenome e a senha padrão metroque.


![](imagens/linux-vpn/10000001000002E00000017010F79EF3.png)


***Insira dados padrão***

---
- Caso tudo dê certo, você estará na tela de trocar a senha. Caso dê algum erro de permissão, peça ajuda ao SysAdmin.

![](imagens/linux-vpn/guia91-41a8c550a8fbe669f1bf6ba9ed9a0c6f.png)

**_Insira seus dados e clique em "Save"._**  
                                              
                                        

  
Pronto! Seu acesso foi validado!


    

# Observações!

  Para evitar enventuais entraves, verifique se suas dependências do
linux estão atualizadas e baixadas. Indico também baixar e instalar o
"Openvpn", por meio do comando ******"sudo apt-get update && sudo
apt-get install openvpn"****** .
