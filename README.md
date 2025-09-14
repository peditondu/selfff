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
```

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
```

### Parte 2: Creiamo la Nostra Rete Privata con Tailscale 🛡️

1.  **Installa Tailscale Ovunque:** Vai sul sito di Tailscale e installa l'app sul tuo **server** e su **tutti i dispositivi** da cui vorrai usare Stremio (telefono, PC, tablet, etc.).

2.  **Accedi con lo Stesso Account:** È fondamentale usare lo stesso account di login su tutti i dispositivi.

3.  **Abilita la MagicDNS ✨:** Vai alla [pagina DNS](https://login.tailscale.com/admin/dns) della tua console Tailscale e assicurati che queste due opzioni siano **abilitate**:
    * MagicDNS
    * HTTPS Certificates

4.  **Trova il Nome del Tuo Server:** Nella [pagina Machines](https://login.tailscale.com/admin/machines), trova il tuo server e copia il suo nome macchina completo (es. `tuoserver.tailxxxx.ts.net`).

Parte 3: Avvio del Progetto dal Repository GitHub 🚀

Assicurati di avere Docker, Docker Compose e Git installati.

### 📥 Clona il repository

Fai il fork del repository selfhost e clonalo sul tuo server.

```bash
# Scegli un percorso, es. la tua home directory
cd ~
# Sostituisci <TUO-UTENTE>/<NOME-REPO> con i dati del tuo fork
git clone [https://github.com/](https://github.com/)<TUO-UTENTE>/<NOME-REPO>.git
cd <NOME-REPO>
```

### 🌐 Crea la rete Docker esterna proxy

Questa rete permette a Nginx di comunicare con gli altri container.

```bash
docker network create proxy
```

Nota: Questo comando va eseguito una sola volta. Se la rete esiste già, Docker mostrerà un errore che puoi ignorare.

### 🛠️ Creazione dei file .env

Per ogni servizio, copia il file .env_example in un nuovo file .env e personalizzalo.

#### 1. .env per MammaMia

```bash
# Esempio: ./mammamia/.env
TMDB_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxx
PROXY=["http://xxxxxxx-rotate:xxxxxxxxx@p.webshare.io:80"]
FORWARDPROXY=http://xxxxxxx-rotate:xxxxxxxx@p.webshare.io:80/
```

#### 2. .env per MediaFlow Proxy

```bash
# Esempio: ./mfp/.env
API_PASSWORD=password_a_scelta
TRANSPORT_ROUTES={"all://*.ichigotv.net": {"verify_ssl": false}, "all://ichigotv.net": {"verify_ssl": false}}
FORWARDED_ALLOW_IPS=*
```

#### 3. .env per StreamV

```bash
# Esempio: ./streamv/.env
MFP_URL="link del tuo mfp" # Potrai modificarlo dopo con l'URL di Nginx
MFP_PSW="la tua password di mfp"
ENABLE_MPD=false
ANIMEUNITY_ENABLED=true
```

#### 4. .env per AIOStreams

```bash
# Esempio: ./AIOStreams/.env
# Queste sono le impostazioni minime
ADDON_ID="il_tuo_id_addon"
BASE_URL=" " # Lo modificherai dopo con l'URL di Nginx
SECRET_KEY= # Generala con: openssl rand -hex 32
ADDON_PASSWORD=password_a_scelta
```

### 🏗️ Build delle immagini e avvio dei container

Questo comando costruirà le immagini e avvierà tutto in background.

```Bash
docker compose up -d --build
```

🧱 Il flag --build forza Docker a eseguire la build anche se l'immagine esiste già.

🔍 Verifica che tutto sia partito

```Bash
docker compose ps
```
Puoi anche consultare i log in tempo reale con docker compose logs -f.

### Parte 4: Il Vigile Urbano del Traffico (Nginx Proxy Manager) 🚦

#### 4.1: Accedi a NPM

Vai su http://IP_DEL_TUO_SERVER:8181.

Email: admin@example.com

Password: changeme (cambiala subito!)

#### 4.2: Il Certificato Magico di Tailscale 📜

Ottieni il certificato dal terminale del server (sostituisci il nome del server):

```Bash
tailscale cert nome-server.tua-tailnet.ts.net
```
Nota per macOS 🍏: Se il comando non stampa il certificato ma crea dei file, visualizzali con:

```Bash
cat "$HOME/Library/Containers/io.tailscale.ipn.macos/Data/nome-server.tua-tailnet.ts.net.key"
cat "$HOME/Library/Containers/io.tailscale.ipn.macos/Data/nome-server.tua-tailnet.ts.net.crt"
```

#### Carica il certificato su NPM:

In NPM, vai su SSL Certificates > Add SSL Certificate > scheda Custom.

Crea due file temporanei sul tuo PC (privato.key e pubblico.crt), incolla il contenuto e caricali.

Dai un nome facile da ricordare (es. "Certificato Tailscale") e salva.

##### 4.3: Smistiamo il Traffico (Proxy & Stream Hosts)

Useremo un Proxy Host per il servizio principale e degli Stream per gli altri.

Configura il Proxy Host per Mammamia (porta 443 standard):

Vai su Hosts -> Proxy Hosts -> Add Proxy Host.

Domain Names: nome-server.tua-tailnet.ts.net.

Forward Hostname / IP: mammamia.

Forward Port: 8080.

Scheda SSL: Seleziona "Certificato Tailscale" e attiva Force SSL.

Salva.

Configura gli Stream per gli altri servizi:

Vai su Hosts -> Streams -> Add Stream.

Per AIOStreams:

Incoming Port: 8001

Forward Hostname / IP: aiostreams

Forward Port: 3000

Per StreamV:

Incoming Port: 8002

Forward Hostname / IP: streamv

Forward Port: 7860

Per MediaFlow Proxy (mfp):

Incoming Port: 8003

Forward Hostname / IP: mfp

Forward Port: 8888

### Parte 5: La Configurazione Finale di Stremio 🎬

Attiva Tailscale sul dispositivo da cui vuoi usare Stremio.

Apri Stremio, vai su Addons (icona 🧩) > I Miei Addon e disinstalla le vecchie versioni.

Reinstalla gli addon usando i nuovi indirizzi sicuri:

https://nome-server.tua-tailnet.ts.net (per Mammamia)

https://nome-server.tua-tailnet.ts.net:8001 (per AIOStreams)

https://nome-server.tua-tailnet.ts.net:8002 (per StreamV)

https://nome-server.tua-tailnet.ts.net:8003 (per MFP)

### Parte 6: Mantenere la Barca a Galla (Aggiornamenti) 🔄

Per aggiornare un servizio da GitHub (es. streamv):

```Bash
docker compose build --pull --no-cache streamv
docker compose up -d
```

Per aggiornare un servizio da un'immagine Docker (es. aiostreams):

```Bash
docker compose pull aiostreams
docker compose up -d
```

### SOS? Sezione Troubleshooting 🆘

L'indirizzo .ts.net non funziona?

Tailscale è attivo sul dispositivo che stai usando?

MagicDNS è abilitato nella console di Tailscale?

Vedo la vecchia grafica dopo un aggiornamento?

Fai un hard-refresh del browser (su Mac: Cmd+Shift+R, su Windows: Ctrl+F5).

Un container è in errore?

Controlla i suoi log: docker compose logs nome_servizio.

Utilizza servizi di IA come Google Gemini o ChatGPT per risolvere i problemi. Il trucco è copiare e incollare gli errori del terminale.

Conclusione 🎉

Congratulazioni! Hai appena costruito un media center personale, privato, sicuro e incredibilmente potente. Ora sei tu il padrone del tuo streaming.
