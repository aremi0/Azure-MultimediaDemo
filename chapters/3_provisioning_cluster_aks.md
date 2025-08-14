# 3. Provisioning di un cluster AKS

Il **pool di sistema** in AKS è dove girano i componenti fondamentali di Kubernetes, come:

* `coredns`: risoluzione DNS interna
* `kube-proxy`: networking
* `metrics-server`: metriche
* `kube-controller-manager`, ecc.

Questi componenti **devono avere risorse minime garantite**, altrimenti il cluster non funziona correttamente. Azure impone che il **pool di sistema** abbia **almeno 2 vCPU e 4 GB RAM**, quindi la VM FreeTier **B1s (1 vCPU, 1 GB)** è troppo piccola.

## ✅ Soluzione consigliata: pool misto (sistema + utente)

### 🔧 Step 1: crea il cluster con pool di sistema su VM `Standard_B2s`

```bash
az aks create \
  --resource-group rg-multimedia \
  --name aks-multimedia \
  --node-count 1 \
  --nodepool-name poolsystem \
  --node-vm-size Standard_B2s \
  --generate-ssh-keys \
  --attach-acr acrmultimedia
```

Questo crea il cluster con un nodo **B2s** (2 vCPU, 4 GB RAM), sufficiente per il sistema. Costo stimato: **~€8–10/mese**, ma coperto dal credito Azure Student.

### 🔧 Step 2: aggiungi un pool utente con `Standard_B2s`
Anche in questo caso non si può usare la VM gratuita `Standard_B1s` perché ritenuta troppo piccola da Azure per poter runnare una Pool di Nodi.  

```bash
az aks nodepool add \
  --resource-group rg-multimedia \
  --cluster-name aks-multimedia \
  --name poolapp \
  --node-count 1 \
  --node-vm-size Standard_B1s \
  --mode User
```

Questo pool è dove **deployi i tuoi servizi** Spring, Redis, Keycloak, ecc...

#### 🔍 Perché è vantaggioso?

| **Pool** | **Tipo VM** | **Costo stimato** | **Uso** | 
| --- | --- | --- | --- | 
| Sistema | B2s | ~€8/mese (coperto da credito) | Componenti Kubernetes | 
| Utente | B2s | ~€8/mese (coperto da credito) | I tuoi microservizi | 

> Così **rispetti i requisiti di Azure**, **minimizzi i costi**, e **mantieni separazione tra sistema e applicazioni** (buona pratica DevOps).

### 🔧 Step 3: puoi scalare il pool utente con autoscaler

```bash
az aks nodepool update \
  --resource-group rg-multimedia \
  --cluster-name aks-multimedia \
  --name poolapp \
  --enable-cluster-autoscaler \
  --min-count 0 \
  --max-count 2
```

### 🔧 Step 4: recupera le credenziali kubeconfig

```bash
az aks get-credentials 
  --resource-group rg-multimedia 
  --name aks-multimedia
```

---

1. Comando 🔧 **Step 1** - Pool di Sistema:

   * `az aks create`: crea un nuovo cluster Kubernetes gestito da Azure.
   * `--resource-group rg-multimedia`: lo inserisce nel gruppo di risorse che hai già creato.
   * `--name aks-multimedia`: assegna il nome *aks-multimedia* al cluster.
   * `--nodepool-name poolsystem`: assegna un nome specifico alla *poolnode* di sistema, anzichè usare quello di default.
   * `--node-count 1`: crea un solo nodo iniziale (una macchina virtuale) nel cluster.
   * `--node-vm-size Standard_B2s`: specifica il tipo di VM (2 vCPU, 4 GB RAM, consuma crediti).
   * `--generate-ssh-keys`: genera automaticamente le chiavi SSH per accedere al nodo.
   * `--attach-acr acrmultimedia`: collega il cluster al tuo Azure Container Registry, così può scaricare le immagini Docker.

2. Comando 🔧 **Step 2** - Pool utente:

   * `az aks nodepool add`: aggiunge un **nuovo pool di nodi** a un cluster AKS esistente.
   * `--resource-group rg-multimedia`: lo inserisce nel gruppo di risorse che hai già creato.
   * `--cluster-name aks-multimedia`: nome del **cluster AKS** a cui aggiungere il pool.
   * `--name poolapp`: nome del **nuovo pool di nodi** (puoi sceglierlo liberamente).
   * `--node-count 1`: crea un solo nodo iniziale (una macchina virtuale) nel cluster.
   * `--node-vm-size Standard_B2s`: specifica il tipo di VM (2 vCPU, 4 GB RAM, consuma crediti).
   * `--gmode User`: specifica che è un **pool utente**, non di sistema.

3. Comando 🔧 **Step 3** - Autoscaler pool utente

   * `--enable-cluster-autoscaler`: abilita l’**autoscaler** sul pool di nodi.
   * Il cluster può **aumentare o ridurre automaticamente** il numero di VM in base al carico.
   * `--name aks-multimedia`: assegna il nome *aks-multimedia* al cluster.
   * `--name poolapp`: aggiorna il **pool di nodi** chiamato `poolapp`.
   * `--min-count 0`: minimo 0 nodo, per evitare consumo di crediti quando è inattivo.
   * `--max-count 2`: massimo 2 nodi.

4. Comando 🔧 **Step 4**:

   * Recupera le **credenziali di accesso** al cluster AKS.
   * Le salva nel file `~/.kube/config`, che è usato da `kubectl` per interagire con il cluster.
   * Dopo questo comando, puoi usare `kubectl get pods`, `kubectl apply`, ecc.

---

### ⚙️ Tipologie di VM disponibili su Azure

| **Serie** | **Uso consigliato** | **Prezzo indicativo (mensile)** | **Note** | 
| --- | --- | --- | --- | 
| **B1s (attuale)** | Progetti di test, sviluppo leggero | ✅ Gratis (750h/mese) | Incluso nel free tier | 
| **B2s** | Maggiore potenza rispetto a B1s | ~$8–10/mese | Ancora burstable, ma con 2 vCPU | 
| **D2s v3** | Generale, produzione leggera | ~$30–40/mese | 2 vCPU, 8 GB RAM | 
| **F2s v2** | Compute-intensive | ~$35–45/mese | 2 vCPU, 4 GB RAM | 
| **E2s v3** | Memory-intensive | ~$50–60/mese | 2 vCPU, 16 GB RAM | 
| **N-Series** | GPU (AI, ML, rendering) | $300+/mese | Solo per carichi speciali | 
| **Spot VMs** | Workload flessibili | ~$1–5/mese | Prezzo variabile, può essere interrotto | 

---

**Puoi spegnere l’intero cluster AKS**, e questo **ferma anche il pool di sistema**, liberando tutte le risorse e **azzerando i costi di calcolo**.

## ✅ Come spegnere e riaccendere il cluster AKS

### 🔻 Spegnere il cluster

```bash
az aks stop \
  --resource-group rg-multimedia \
  --name aks-multimedia
```

### 🔺 Riaccendere il cluster

```bash
az aks start \
  --resource-group rg-multimedia \
  --name aks-multimedia
```

> Questo **sospende sia il control plane che tutti i nodi**, inclusi quelli del pool di sistema. Lo stato del cluster viene **preservato fino a 12 mesi**, quindi puoi riprendere da dove avevi lasciato.

## 🔍 1. Verificare lo stato del cluster AKS

### ✅ Numero di nodi usati dalla PoolNode `poolapp` (0 spenta, 1-2 nodi accesa)

```bash
az aks nodepool show --resource-group rg-multimedia --cluster-name aks-multimedia --name poolapp --query "count"
```

### ✅ Elenco dei cluster e stato di provisioning

```bash
az aks list -o table
```

### ✅ Stato di alimentazione (Running/Stopped)

```bash
az aks list --query "[].{Name:name, Power:powerState.code}" -o table
```

> Questo ti mostra se il cluster è **running** o **stopped**, utile per verificare se consuma risorse.

## 🧱 2. Verificare lo stato dei nodi

Assicurati di avere configurato `kubectl` con:

```bash
az aks get-credentials --resource-group rg-multimedia --name aks-multimedia
```

Poi usa:

```bash
kubectl get nodes
```

> Ti mostra ogni nodo con stato (`Ready`, `NotReady`, ecc.), versione di Kubernetes, e ruolo.

## 📦 3. Verificare lo stato dei pod

```bash
kubectl get pods -A
```

> Mostra tutti i pod in tutti i namespace, utile per capire se ci sono carichi attivi.
```

Se vuoi, posso anche generarti un file `.md` da scaricare in locale. Fammi sapere!