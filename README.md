# Migrazione del progetto 'MultimediaDemo' su cloud Azure AKS e CI/CD con GitHub Actions

[Riassunto dei comandi usati](riassunto_comandi.md)

0. [Contesto](./chapters/0_contesto.md)
1. [Creare un account Azure e preparare l’ambiente locale](./chapters/1_creazione_account_configurazione_ambiente_locale.md)
2. [Creare Resource Group (SR) e Azure Container Registry (ACR)](./chapters/2_creazione_sr_creazione_acr.md)
3. [Provisiong cluster AKS in modalità mista](./chapters/3_provisioning_cluster_aks.md)
4. []()
5. []()
6. []()
7. []()
8. []()
9. []()
10. []()
11. []()

---
---
---

# work in progress -> to elaborate better

## 4. Preparare immagini Docker e push su ACR

1. In locale, builda ogni immagine:  
   ```bash
   docker build -t acrmultimedia.azurecr.io/stream-mp3:latest ./StreamMp3
   ```  
2. Effettua il login OIDC da GitHub Actions (in pipeline non userai credenziali statiche). In locale puoi abilitare l’admin:  
   ```bash
   az acr update --name acrmultimedia --admin-enabled true
   az acr login --name acrmultimedia
   ```  
3. Pusha le immagini di prova (puoi saltare i tag semantici nella fase iniziale):  
   ```bash
   docker push acrmultimedia.azurecr.io/stream-mp3:latest
   ```

---

## 5. Convertire i servizi in manifest Kubernetes

Per ciascun servizio:

- **Deployment**: replica, resource requests/limits, variabili d’ambiente, `volumeMounts` per secrets o PVC.  
- **Service**: `ClusterIP` o `LoadBalancer` secondo necessità.  
- **ConfigMap/Secret**: sposta le variabili sensibili (`POSTGRES_PASSWORD`, TLS key/cert) in Secret.  
- **PersistentVolumeClaim**: per Postgres e Kafka (se vuoi persistenza).

Suggerimento da neofita: inizia con Helm charts pre-confezionati di Bitnami/JetStack:

- Kafka senza ZooKeeper: `helm repo add bitnami https://charts.bitnami.com/bitnami`  
- Postgres: `helm install pg bitnami/postgresql`  
- Keycloak: `helm repo add codecentric https://codecentric.github.io/helm-charts/`  
- Redis: `helm install redis bitnami/redis`  
- Ingress NGINX: `helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx`

---

## 6. Configurare ingress TLS e routing

1. Installa `cert-manager`:  
   ```bash
   kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/latest/download/cert-manager.yaml
   ```  
2. Crea un Issuer per Let’s Encrypt (staging per test iniziali, poi produzione).  
3. Deploya l’Ingress Controller:  
   ```bash
   helm install nginx-ingress ingress-nginx/ingress-nginx \
     --set controller.publishService.enabled=true
   ```  
4. Definisci un `Ingress` che instrada `/mp3`, `/pdf`, `/keycloak`, ecc., e abilita TLS con Annotazioni cert-manager.

---

## 7. Deploy dei microservizi Spring e del log-forwarder

1. Crea namespaces separati (`dev`, `prod` in futuro).  
2. Applica i manifest generati o i chart Helm custom per ciascun service.  
3. Verifica con `kubectl get pods,svc,ing` e log con `kubectl logs`.

---

## 8. Configurare GitHub Actions per CI/CD

Aggiungi nella root `.github/workflows/ci-cd.yml`:

```yaml
name: CI/CD

on:
  push:
    branches: [ main ]

permissions:
  id-token: write
  contents: read
  packages: write

jobs:
  build-push-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Azure Login
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Build and Push Images
        uses: azure/docker-login@v1
        with:
          login-server: acrmultimedia.azurecr.io

      - run: |
          docker build -t acrmultimedia.azurecr.io/stream-mp3:${{ github.sha }} ./StreamMp3
          docker push acrmultimedia.azurecr.io/stream-mp3:${{ github.sha }}
          # ripeti per gli altri servizi

      - name: Deploy to AKS
        run: |
          az aks get-credentials --resource-group rg-multimedia --name aks-multimedia
          helm upgrade --install stream-mp3 ./k8s/stream-mp3 \
            --set image.tag=${{ github.sha }}
          # ripeti helm upgrade per ogni chart
```

1. Registra in GitHub i secrets: `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`.  
2. Configura un Service Principal e assegna il ruolo `Azure Kubernetes Service Cluster User Role` e `AcrPull` al tuo SPN.

---

## 9. Verifica e rollback automatico

- Aggiungi health/readiness probes nei Deployment Spring.  
- Inserisci step di smoke test post-deploy (es. `curl` o piccoli test JUnit in GitHub Actions).  
- Configura Helm con `--atomic` per rollback automatico in caso di deploy fallito.

---

## 10. Monitoraggio, logs e costi

- Installa Prometheus/Grafana via Helm per metriche.  
- Abilita Azure Monitor Container Insights per log e diagnosi.  
- Imposta un budget in Azure Cost Management e avvisi e-mail.

---

## 11. Prossimi passi

Una volta completata questa base, puoi approfondire:

- Infrastructure as Code con Terraform o Bicep per automatizzare la creazione di RG, ACR, AKS, Issuer, ecc.  
- Canary/Blue-Green deployment con Argo Rollouts o Flagger.  
- Sicurezza avanzata: NetworkPolicy, PodSecurityPolicy, Azure Policy per Kubernetes.  
- Environment dinamici per feature branch usando Helmfile o GitHub Environments.  
- Service mesh (Istio/Linkerd) per resilienza, mTLS e telemetria avanzata.

Se vuoi, posso fornirti esempi di manifest Kubernetes, Helm chart customizzati e file Terraform/Bicep completi, oltre a snippet più dettagliati di GitHub Actions. Fammi sapere su quale parte preferisci approfondire per prima!