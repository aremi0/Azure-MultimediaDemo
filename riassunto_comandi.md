
1. Crea il Resource Group:
    ```bash
    az group create 
        --name rg-multimedia 
        --location swedencentral
    ```

2. Crea magazzino immagini (ACR):
    ```bash
    az acr create 
        --resource-group rg-multimedia 
        --name acrmultimedia 
        --sku Basic 
        --admin-enabled false
    ```

3. Crea cluster di Sistema orchestrato da K8s (AKS):
    ```bash
    az aks create \
        --resource-group rg-multimedia \
        --name aks-multimedia \
        --nodepool-name poolsystem \
        --node-count 1 \
        --node-vm-size Standard_B2s \
        --generate-ssh-keys \
        --attach-acr acrmultimedia
    ```

4. Crea nuovo poolnode per gestire multimedia-project:
    ```bash
    az aks nodepool add \
        --resource-group rg-multimedia \
        --cluster-name aks-multimedia \
        --name poolapp \
        --node-count 1 \
        --node-vm-size Standard_B2s \
        --mode User
    ```

5. Salvare le credenziali del cluster AKS appena creato in kubectl
    ```bash
    az aks get-credentials \
        --resource-group rg-multimedia \
        --name aks-multimedia
    ```

6. Attivare l’autoscaler al poolnode che gestisce il numero di nodi da allocare per l’app multimedia-project:
    ```bash
    az aks nodepool update \
        --resource-group rg-multimedia \
        --cluster-name aks-multimedia \
        --name poolapp \
        --enable-cluster-autoscaler \
        --min-count 0 \
        --max-count 2
    ```

7. ## 💰 Monitoring

    * Conta il numero di nodi allocati per il nodepool poolapp
        ```bash
        az aks nodepool show \
            --resource-group rg-multimedia \
            --cluster-name aks-multimedia \
            --name poolapp \
            --query "count"
        ```

    * Spegnere il cluster AKS per risparmiare crediti:
        ```bash
        az aks stop \
            --resource-group rg-multimedia \
            --name aks-multimedia
        ```

    * Accendere il cluster AKS quando devo lavorarci:
        ```bash
        az aks start \
            --resource-group rg-multimedia \
            --name aks-multimedia
        ```

8. punto
    ```bash
    codice
    ```

9. punto
    ```bash
    codice
    ```

10. punto
    ```bash
    codice
    ```

11. punto
    ```bash
    codice
    ```

12. punto
    ```bash
    codice
    ```