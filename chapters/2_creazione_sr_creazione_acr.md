# 2. Creare Resource Group e Azure Container Registry (ACR)

## 🧱 Cos'è un Resource Group?

Un **Resource Group** è come una cartella logica in Azure dove puoi raggruppare e gestire tutte le risorse correlate a un progetto: container, database, reti, ecc. Serve per organizzare, monitorare, e applicare regole (come il controllo dei costi o i permessi) in modo centralizzato.

#### ✅ Comando:

```bash
az group create 
  --name rg-multimedia 
  --location swedencentral
```

#### 🔍 Cosa fa:

- `az group create`: crea un nuovo gruppo di risorse.
- `--name rg-multimedia`: lo chiami “rg-multimedia” (puoi scegliere qualsiasi nome).
- `--location swedencentral`: indica il datacenter Azure in Germania centrale (vicino a te, quindi con meno latenza).  

***P.s.** uso `swedencentral` anziché la più comune `westeurope` semplicemente perché costretto dalle policy ristrette del profilo Azure Student.*

---

### 📦 Cos'è Azure Container Registry (ACR)?

ACR è un **registro privato di immagini Docker**. È come Docker Hub, ma ospitato su Azure. Ti permette di:

- Archiviare le tue immagini container.
- Usarle per distribuire applicazioni su Kubernetes, App Services, ecc.
- Automatizzare build e aggiornamenti.

#### ✅ Comando:

```bash
az acr create 
  --resource-group rg-multimedia 
  --name acrmultimedia 
  --sku Basic 
  --admin-enabled false
```

#### 🔍 Cosa fa:

- `az acr create`: crea un nuovo registro container.
- `--resource-group rg-multimedia`: lo collega al gruppo di risorse che hai appena creato.
- `--name acrmultimedia`: il nome del tuo registro (deve essere univoco a livello globale).
- `--sku Basic`: scegli il piano “Basic” (perfetto per studenti e piccoli progetti).
- `--admin-enabled false`: disabilita l’accesso con credenziali admin (più sicuro, si usa l’autenticazione con identità gestite o service principal).

---

### 🧠 Perché è importante?

Questi due passaggi sono le **fondamenta**:

- Il Resource Group è il contenitore logico.
- L’ACR è il magazzino delle immagini che userai per distribuire la tua app su Kubernetes (AKS) o altri servizi.

---

### 💰 Azure Student: cosa include?

Il profilo Azure for Students ti offre **$100 di credito gratuito per 12 mesi**, senza bisogno di carta di credito. Include anche accesso gratuito a diversi servizi, ma non tutto è gratuito.

#### 🔍 Costi delle risorse che stai creando

| Risorsa | Prezzo con SKU scelto | Incluso nel piano Student? | Note importanti |
|--------|------------------------|-----------------------------|------------------|
| **Resource Group** | ✅ Gratis | ✅ Sì | È solo un contenitore logico |
| **Azure Container Registry (ACR)** | 💲 Circa €0.003/GB/giorno con SKU Basic | ❌ No (non incluso) | Consuma credito se usi spazio |

#### 🧮 Stima per il tuo caso

Supponiamo:

- Hai **2 immagini personalizzate**.
- Ogni immagine pesa **300 MB** (esempio realistico).
- Totale: **600 MB = 0.6 GB**

#### 🔢 Calcolo mensile:

- €0.003 × 0.6 GB × 30 giorni ≈ **€0.054 al mese**

> Quindi: **circa 5 centesimi di euro al mese**. Un costo **quasi trascurabile**, ma che **consuma i crediti Azure Student**.
