# Il Tuo Stremio Self-Hosted 🚀
### Sicuro con Docker 🐳, Privato con Tailscale 🛡️, Accessibile Ovunque 🌍

Ciao a tutti, amici nerd! 👋

Siete stanchi di configurazioni complicate e volete il pieno controllo del vostro streaming? Perfetto! Questa guida vi accompagnerà passo dopo passo nella creazione di un'istanza Stremio-addons privata, usando la magia di Docker e la sicurezza di Tailscale.

L'obiettivo finale? Avere il vostro "Netflix" personale, accessibile in modo sicuro dal vostro telefono in 4G o dal PC di un amico, **senza mai esporre la vostra rete di casa su Internet**.

Basta chiacchiere, rimbocchiamoci le maniche!

---

### ⚠️ Disclaimer - Nota Importante
> Questa guida e le configurazioni in essa contenute sono state create a scopo puramente **educativo e sperimentale**. L'obiettivo è esplorare le possibilità del self-hosting e della gestione di reti private.
> 
> Si prega di leggere attentamente i seguenti punti prima di procedere:
> * **Responsabilità dell'Utente:** L'utilizzo degli strumenti e delle procedure descritte è a totale **rischio e pericolo dell'utente**. L'autore di questa guida non si assume alcuna responsabilità per le azioni intraprese dai lettori o per il corretto funzionamento del software installato.
> * **Rispetto del Copyright:** Gli strumenti menzionati in questa guida possono essere utilizzati per accedere a contenuti protetti da copyright. È responsabilità esclusiva dell'utente assicurarsi di avere il **diritto legale** di accedere, visualizzare o distribuire tali contenuti secondo le leggi vigenti nel proprio paese. L'autore condanna fermamente la pirateria e invita a utilizzare questa configurazione solo con materiale di cui si detengono i diritti o che sia di pubblico dominio.
> * **Nessuna Garanzia:** La guida è fornita "così com'è", senza alcuna garanzia di funzionamento, accuratezza, sicurezza o idoneità a uno scopo specifico.
> * **Limitazione di Responsabilità:** L'autore non potrà essere ritenuto responsabile per eventuali danni diretti o indiretti, perdita di dati, problemi di sicurezza, o conseguenze legali derivanti dall'applicazione delle informazioni contenute in questa guida.
> 
> **Procedendo con l'installazione, l'utente dichiara di aver compreso e accettato questi termini e si assume la piena responsabilità del proprio operato.**

---

### Ringraziamenti Speciali 🙏
La creazione di questa guida non sarebbe stata possibile senza il lavoro e l'ispirazione di altri membri della community. Un ringraziamento particolare va a:
* **[nihon77](https://github.com/nihon77):** Per aver creato la guida originale basata su DuckDNS e Port Forwarding. Il suo eccellente lavoro è stato la fonte di ispirazione e il punto di partenza fondamentale per sviluppare questa versione alternativa incentrata sulla sicurezza con Tailscale.
* **[nzo66](https://github.com/nzo66):** Per il suo prezioso contributo nella creazione e manutenzione della versione modificata di MediaFlow Proxy. Questo componente è una parte cruciale dello stack per molti utenti italiani.

---

### Indice dei Contenuti 🗺️
1.  [Cosa Ti Serve Prima di Iniziare? 🛠️](#cosa-ti-serve-prima-di-iniziare-️)
2.  [Parte 1: Installiamo le Basi (Docker & Git)](#parte-1-installiamo-le-basi-docker--git)
3.  [Parte 2: Creiamo la Nostra Rete Privata con Tailscale 🛡️](#parte-2-creiamo-la-nostra-rete-privata-con-tailscale-️)
4.  [Parte 3: Avvio del Progetto dal Repository GitHub 🚀](#parte-3-avvio-del-progetto-dal-repository-github-)
5.  [Parte 4: Il Vigile Urbano del Traffico (Nginx Proxy Manager) 🚦](#parte-4-il-vigile-urbano-del-traffico-nginx-proxy-manager-)
6.  [Parte 5: La Configurazione Finale di Stremio 🎬](#parte-5-la-configurazione-finale-di-stremio-)
7.  [Parte 6: Mantenere la Barca a Galla (Aggiornamenti) 🔄](#parte-6-mantenere-la-barca-a-galla-aggiornamenti-)
8.  [SOS? Sezione Troubleshooting 🆘](#sos-sezione-troubleshooting-)
9.  [Conclusione 🎉](#conclusione-)

---

### Cosa Ti Serve Prima di Iniziare? 🛠️
* **Un Serverino 🖥️:** Un PC, un Mac Mini, un vecchio laptop, un Raspberry Pi 4 (o superiore), o un NAS che possa rimanere sempre acceso.
* **Sistema Operativo:** Consigliato un Linux basato su Debian (es. Ubuntu Server 22.04 LTS 🐧). Se usi macOS 🍏, i passaggi sono identici una volta installato Docker Desktop.
* **Account Gratuiti:**
    * Un account [GitHub](https://github.com/) 🐙
    * Un account [Tailscale](https://tailscale.com/) ✨
* **Fork dei Repository:** Fai un "Fork" di questi repository sul tuo account GitHub. Ti servirà per clonare il codice.
    * `https://github.com/xlab992/selfhost.git`
    * `https://github.com/nzo66/mediaflow-proxy.git`

---

### Parte 1: Installiamo le Basi (Docker & Git)
Questi comandi sono per **Ubuntu/Debian**.

#### 1.1: Installazione di Git 🔧
```bash
sudo apt update
sudo apt install git -y

### 1.2: Installazione di Docker e Docker Compose 🐳
Installiamo la versione più recente e ufficiale.

```bash
# Rimuovi eventuali versioni precedenti
sudo apt remove docker docker-engine docker.io containerd runc -y

# Aggiorna l'elenco dei pacchetti
sudo apt update

# Installa i pacchetti richiesti
sudo apt install -y ca-certificates curl gnupg lsb-release

# Aggiungi la chiave GPG ufficiale di Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL [https://download.docker.com/linux/$](https://download.docker.com/linux/$)(. /etc/os-release; echo "$ID")/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Aggiungi il repository Docker
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] [https://download.docker.com/linux/$](https://download.docker.com/linux/$)(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Installa Docker, Docker Compose e altri componenti
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Aggiungi il tuo utente al gruppo docker per evitare di usare 'sudo'
sudo usermod -aG docker $USER

# Applica le modifiche (richiede un nuovo login o questo comando)
newgrp

# Verifica che Docker funzioni
docker run hello-world
