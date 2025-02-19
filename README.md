# Guia para Instalação de Drivers NVIDIA no Debian 12


Este guia ajudará você a verificar se seu hardware possui uma placa gráfica NVIDIA, identificar o driver recomendado e instalar os drivers corretamente. Também abordaremos como lidar com o Secure Boot, que pode impedir o carregamento dos drivers.


---
# ⚠️ Aviso de Responsabilidade

A execução deste tutorial é de inteira responsabilidade do usuário. Certifique-se de entender cada etapa antes de prosseguir. Embora este guia tenha sido testado no Debian 12, ele pode não funcionar corretamente em outras distribuições ou versões do sistema. Recomenda-se fazer backup dos dados importantes antes de iniciar.

---

## 1. Verificando se seu hardware possui uma placa gráfica NVIDIA

Para verificar se seu sistema possui uma placa gráfica NVIDIA, execute o seguinte comando no terminal:

```bash
lspci | grep -i nvidia
```
**Resultado esperado:**
```bash
01:00.0 VGA compatible controller: NVIDIA Corporation GP106GL [Quadro P2200] (rev a1)
01:00.1 Audio device: NVIDIA Corporation GP106 High Definition Audio Controller (rev a1)
```
Se você vir uma saída semelhante, significa que seu sistema possui uma placa gráfica NVIDIA.

---

## 2. Identificando o driver recomendado

Para identificar o driver mais adequado para seu hardware, instale o utilitário `nvidia-detect` e execute-o:

```bash
sudo apt update
sudo apt install nvidia-detect linux-headers-amd64
nvidia-detect
```
**Resultado esperado:**  
O utilitário recomendará o driver apropriado para sua placa gráfica, por exemplo:

```bash
Detected NVIDIA GPUs:
01:00.0 VGA compatible controller: NVIDIA Corporation GP106GL [Quadro P2200] (rev a1)

It is recommended to install the following package:
nvidia-driver
```

## 3. Instalando os drivers NVIDIA

Com base na recomendação do `nvidia-detect`, instale o driver apropriado. Por exemplo:

```bash
sudo apt install nvidia-driver firmware-misc-nonfree
```
Após a instalação, reinicie o sistema para aplicar as alterações:

```bash
sudo reboot
```

## 4. Lidando com o Secure Boot

O Secure Boot pode impedir que os drivers NVIDIA sejam carregados durante a inicialização. Siga os passos abaixo para resolver esse problema.

### 4.1 Verificando o estado do Secure Boot

Verifique se o Secure Boot está ativado:

```bash
sudo apt install mokutil
sudo mokutil --sb-state
```

Se o Secure Boot estiver ativado, você precisará assinar os módulos do kernel manualmente.

### 4.2 Gerando chaves para assinar os módulos

1. Gere uma chave MOK (Machine Owner Key):
    
```bash
sudo apt install openssl
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.key -out MOK.pem -nodes -days 36500 -subj "/CN=Custom Secure Boot Key/"
openssl x509 -in MOK.pem -outform DER -out MOK.der
```
2. Importe a chave MOK:
```bash    
    sudo mokutil --import MOK.der
```
3. Siga as instruções na tela para definir uma senha. Essa senha será solicitada na próxima inicialização.
4. Reinicie o sistema e complete o processo de importação da chave MOK.
    
### 4.3 Assinando os módulos do kernel

Liste os módulos:
```bash
ls /lib/modules/$(uname -r)/updates/dkms/ 
```
Assine os módulos do kernel NVIDIA com a chave gerada como no seguinte exemplo:

```bash
sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 MOK.key MOK.pem /lib/modules/$(uname -r)/updates/dkms/nvidia-current-drm.ko
```
**Importante**
Ao reinciar a EFI pode pedir a confirmação e a senha anteriormente inserida.

## 5. Verificando a instalação dos drivers

Após reiniciar o sistema, verifique se os drivers NVIDIA estão funcionando corretamente:

```bash
nvidia-smi
```
**Resultado esperado:**  


```bash
Uma tabela mostrando informações sobre a placa gráfica NVIDIA, como uso de GPU, memória e processos em execução.

```

## 6. Considerações finais

- **Blacklist de drivers open-source:** Para evitar conflitos, é recomendado adicionar os drivers open-source (como `nouveau`) à blacklist. Isso geralmente é feito automaticamente durante a instalação dos drivers NVIDIA.
    
- **Atualizações:** Mantenha seus drivers atualizados para obter melhor desempenho e compatibilidade com novos softwares.
    
