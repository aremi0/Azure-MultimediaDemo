# 0. Contesto

Sto sviluppando una infrastruttura software con docker, che comprende un punto di ingresso con un container nginx che funge da reverse proxy che centralizza https, un container keycloak per gestire oauth2, e poi tutta una serie mi container di microservizi Spring compresi un container per l'api gateway che usa spring cloud webflux ed un'altro eurekaserver per il routing dinamico. Lo scopo è puramente formativo, ma ciò non vuol togliere nulla alla professionalità del progetto all'alto livello di attenzione nei dettagli di sicurzza, prestazioni e scalabilità tipici di un qualunque progetto in ambiente di esercizio/produzione. Fatta questa premessa, mi piacerebbe molto imparare le tecniche di DevOps e perciò pensavo di migrare questa infrastruttura in un qualche servizio cloud che sia gratuito (sarei disposto anche a pagare una cifra non elevata in realtà). In realtà non so benissimo come funzionano le tecnologie di DevOps, quello che so è che voglio creare un pipeline CI/CD completa e imparare.

Il mio sogno è di imparare ad usare le più famose (Azure o AWS ad esempio). A me piacerebbe vedere K8s e Github Actions per la CI/CD. Il mio bugdet massimo è il minimo che posso spendere. Prevedo che implementerò 2 o 3 microservizi Spring al massimo, uno per lo streaming di file mp3 e un altro per lo streaming di file pdf, vorrei però includere sistemi di caching con redis. Il traffico atteso è basso, perchè lo userò solamente io per studiare e sviluppare

## Raccomandazione: partire con Azure per K8s e GitHub Actions

Per un ambiente di studio con budget minimo, Azure offre un vantaggio chiave: il controllo piano di AKS è gratuito e include 750 ore al mese di VM Standard_B1s (1 vCPU, 1 GB RAM) per 12 mesi. In più ricevi 100 USD di crediti iniziali e integrazione nativa con GitHub Actions. Con questi elementi potrai esplorare Kubernetes reale senza costi fissi sul control plane.

### Stima dei costi mensili

| **Risorsa**           | **Tipologia**               | **Costo stimato**     |
|-----------------------|-----------------------------|------------------------|
| VM Standard_B1s       | 1 nodo AKS (750 h gratis)   | 0 USD                  |
| AKS control plane     | gratuito                    | 0 USD                  |
| ACR storage           | 0–2 GiB gratis              | 0 – 0.33 USD/Mese      |
| Traffico e LoadBalancer | ambiente dev, traffico basso | trascurabile        |

**Totale:** tra 0 e ~1 USD/mese finché rimani nei free tiers e nel credito iniziale.
