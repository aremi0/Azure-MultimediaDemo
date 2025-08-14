# 1. Creare un account Azure e preparare l’ambiente locale

1. Vai su [https://azure.microsoft.com/free](https://azure.microsoft.com/free) e registra il tuo account gratuito. Otterrai 200 USD di credito iniziale.

2. Installa Azure CLI, apri *powershell* come amministratore:

   ```bash
   $ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://aka.ms/installazurecliwindowsx64 -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; Remove-Item .\AzureCLI.msi
   ```

3. Autentica la CLI:

   ```bash
   az login
   ```

4. Scarica `kubectl` e `helm` e dragga gli eseguibili nella cartella D:\Tools\kubectl e D:\Tools\helm

5. Aggiungi i due path del punto (4) al %PATH% d'ambiente per l'utente