Il Tuo Stremio Self-Hosted 🚀
Sicuro con Docker 🐳, Privato con Tailscale 🛡️, Accessibile Ovunque 🌍
Ciao a tutti, amici nerd! 👋
Siete stanchi di configurazioni complicate e volete il pieno controllo del vostro streaming? Perfetto! Questa guida vi accompagnerà passo dopo passo nella creazione di un'istanza Stremio-addons privata, usando la magia di Docker e la sicurezza di Tailscale.
L'obiettivo finale? Avere il vostro "Netflix" personale, accessibile in modo sicuro dal vostro telefono in 4G o dal PC di un amico, senza mai esporre la vostra rete di casa su Internet.
Basta chiacchiere, rimbocchiamoci le maniche!
⚠️ Disclaimer - Nota Importante
Questa guida e le configurazioni in essa contenute sono state create a scopo puramente educativo e sperimentale. L'obiettivo è esplorare le possibilità del self-hosting e della gestione di reti private.
Si prega di leggere attentamente i seguenti punti prima di procedere:
•	Responsabilità dell'Utente: L'utilizzo degli strumenti e delle procedure descritte è a totale rischio e pericolo dell'utente. L'autore di questa guida non si assume alcuna responsabilità per le azioni intraprese dai lettori o per il corretto funzionamento del software installato.
•	Rispetto del Copyright: Gli strumenti menzionati in questa guida possono essere utilizzati per accedere a contenuti protetti da copyright. È responsabilità esclusiva dell'utente assicurarsi di avere il diritto legale di accedere, visualizzare o distribuire tali contenuti secondo le leggi vigenti nel proprio paese. L'autore condanna fermamente la pirateria e invita a utilizzare questa configurazione solo con materiale di cui si detengono i diritti o che sia di pubblico dominio.
•	Nessuna Garanzia: La guida è fornita "così com'è", senza alcuna garanzia di funzionamento, accuratezza, sicurezza o idoneità a uno scopo specifico. Le configurazioni potrebbero diventare obsolete o richiedere manutenzione non descritta in questo documento.
•	Limitazione di Responsabilità: L'autore non potrà essere ritenuto responsabile per eventuali danni diretti o indiretti, perdita di dati, problemi di sicurezza, o conseguenze legali derivanti dall'applicazione delle informazioni contenute in questa guida.
Procedendo con l'installazione, l'utente dichiara di aver compreso e accettato questi termini e si assume la piena responsabilità del proprio operato.
Ringraziamenti Speciali 🙏
La creazione di questa guida non sarebbe stata possibile senza il lavoro e l'ispirazione di altri membri della community. Un ringraziamento particolare va a:
•	nihon77: Per aver creato la guida originale basata su DuckDNS e Port Forwarding. Il suo eccellente lavoro è stato la fonte di ispirazione e il punto di partenza fondamentale per sviluppare questa versione alternativa incentrata sulla sicurezza con Tailscale.
•	nzo66: Per il suo prezioso contributo nella creazione e manutenzione della versione modificata di MediaFlow Proxy. Questo componente è una parte cruciale dello stack per molti utenti italiani e il suo impegno nel renderlo accessibile è un grande aiuto per tutta la community.
 
Cosa Ti Serve Prima di Iniziare? 🛠️
•	Un Serverino 🖥️: Un PC, un Mac Mini, un vecchio laptop, un Raspberry Pi 4 (o superiore), o un NAS che possa rimanere sempre acceso.
•	Sistema Operativo: Consigliato un Linux basato su Debian (es. Ubuntu Server 22.04 LTS 🐧). Se usi macOS 🍏, i passaggi sono identici una volta installato Docker Desktop.
•	Account Gratuiti:
o	Un account GitHub 🐙
o	Un account Tailscale ✨
•	Fai il fork di questi repository su GitHub: 
o	https://github.com/xlab992/selfhost.git 
o	https://github.com/nzo66/mediaflow-proxy.git 
 
Parte 1: Installiamo le Basi (Docker & Git)
Questi comandi sono per Ubuntu/Debian.
1.1: Installazione di Git 🔧
Bash
sudo apt update
sudo apt install git -y
1.2: Installazione di Docker e Docker Compose 🐳
Installiamo la versione più recente e ufficiale.
📥 Installazione Docker
# 🔁 Rimuovi eventuali versioni precedenti
sudo apt remove docker docker-engine docker.io containerd runc

# 🔄 Aggiorna l’elenco dei pacchetti
sudo apt update

# 📦 Installa i pacchetti richiesti per aggiungere il repository Docker
sudo apt install -y ca-certificates curl gnupg lsb-release

# 🗝️ Aggiungi la chiave GPG ufficiale di Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 📥 Aggiungi il repository Docker alle fonti APT
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 🐳 Installa Docker, Docker Compose e altri componenti
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 👤 Aggiungi il tuo utente al gruppo docker
sudo usermod -aG docker $USER

# ⚠️ Per applicare il cambiamento, esegui il logout/login oppure:
newgrp

# ✅ Verifica che Docker funzioni correttamente
docker run hello-world

 
Parte 2: Creiamo la Nostra Rete Privata con Tailscale 🛡️
1.	Installa Tailscale Ovunque: Vai sul sito di Tailscale e installa l'app sul tuo server e su tutti i dispositivi da cui vorrai usare Stremio (telefono, PC, tablet, etc.).
2.	Accedi con lo Stesso Account: È fondamentale usare lo stesso account di login su tutti i dispositivi per farli entrare nella stessa rete.
3.	Abilita la MagicDNS ✨: Vai alla pagina DNS della tua console Tailscale e assicurati che queste due opzioni siano abilitate:
o	MagicDNS
o	HTTPS Certificates
4.	Trova il Nome del Tuo Server: Nella pagina Machines, trova il tuo server e copia il suo nome macchina completo (es. tuoserver.tailxxxx.ts.net). Tienilo a portata di mano!
 
Parte 3: 🚀 Avvio del progetto dal repository GitHub
Il progetto è contenuto in un repository GitHub che include un file docker-compose.yml preconfigurato. Alcuni servizi costruiranno automaticamente le immagini Docker a partire da Dockerfile remoti ospitati su GitHub.
🔧 Prerequisiti
Assicurati di avere:
•	Docker e Docker Compose installati (vedi sezione precedente)
•	Git installato (sudo apt install git se non lo hai)
📥 Clona il repository
Fai il fork del repository selfhost col tuo account Github e dopodiché clona il tuo repo sul tuo server
cd ~ # percorso dove vuoi clonare il repo es. cd /users/tuonome/documenti
git clone https://github.com/tuo-utente/tuo-repo.git
cd tuo-repo
🌐 Crea la rete Docker esterna proxy
Se non l'hai già fatto, crea la rete che verrà utilizzata da Nginx Proxy Manager e dagli altri container per comunicare tra loro:
docker network create proxy
🔁 Questo comando va eseguito una sola volta. Se la rete esiste già, Docker mostrerà un errore che puoi ignorare in sicurezza.

🛠️ Creazione dei file .env per MammaMia, MediaFlow Proxy, StreamV, AIOStreams e docker-duckdns
In ogni sotto cartella di questo progetto è presente un file .env_example con tutte le chiavi necessarie per il corretto funzionamento dei vari moduli. Per ogni modulo copiare e rinominare il file .env_example in .env. I vari .env dovranno essere modificati in base alle vostre specifiche configurazioni.
1. .env per MammaMia Per configurare il plugin MammaMia è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.
📄 Esempio: ./mammamia/.env
# File .env per il plugin mammamia
TMDB_KEY=xxxxxxxxxxxxxxxx
PROXY=["http://xxxxxxx-rotate:xxxxxxxxx@p.webshare.io:80"]
FORWARDPROXY=http://xxxxxxx-rotate:xxxxxxxx@p.webshare.io:80/
2. .env per MediaFlow Proxy Per configurare il modulo Media Flow Proxy è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.
📄 Esempio: ./mfp/.env
API_PASSWORD=password
TRANSPORT_ROUTES={"all://*.ichigotv.net": {"verify_ssl": false}, "all://ichigotv.net": {"verify_ssl": false}}
# Trust all Docker IPs (less secure but simpler for development)
FORWARDED_ALLOW_IPS=*
# or Trust the Docker network range (when nginx and mediaflow-proxy are in same docker network)
#FORWARDED_ALLOW_IPS=172.20.0.0
3. .env per StreamV Per configurare il plugin StreamV è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.
📄 Esempio: ./streamv/.env
#############################################
# StreamViX Environment Configuration
# Copy this file to .env and adjust values.
# All variables are optional; defaults are applied in code.
#############################################

# MediaFlow Proxy
MFP_URL="link del tuo mfp. Potrai modificarlo successivamente con quello generato su nginx"
MFP_PSW="la tua pass"

# Feature toggles (true/false)
ENABLE_MPD=false
ANIMEUNITY_ENABLED=true
ANIMESATURN_ENABLED=true
ANIMEWORLD_ENABLED=true

# Dynamic events extractor behavior
# Set FAST_DYNAMIC=1 to bypass extractor and use direct URLs from dynamic_channels.json
FAST_DYNAMIC=0
# Concurrency for extractor resolution of dynamic events (1-50). Also used as CAP of dynamic links processed.
DYNAMIC_EXTRACTOR_CONC=10

# TMDB API key override (leave empty to use default)
TMDB_API_KEY=


#############################################
# Notes:
# - If both FAST_DYNAMIC=1 and DYNAMIC_EXTRACTOR_CONC are set, FAST wins (extractor skipped).
# - Tiered priority: (it|ita|italy) first, then (italian|sky|tnt|amazon|dazn|eurosport|prime|bein|canal|sportitalia|now|rai)
# - ENABLE_MPD defaults to false if unset.
# - Anime providers default to enabled unless explicitly set to false.
#############################################

4. .env per AIOStreams Per configurare il plugin AIOStreams è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.
📄 Esempio: ./AIOStreams/.env
#queste sono le impostazioni minime per il corretto funzionamento del plugin
ADDON_ID=" "
BASE_URL=
SECRET_KEY= (può essere generata con comando da terminale: openssl rand -hex 32)
ADDON_PASSWORD=password_a_scelta

🛑 Attenzione alla sicurezza: imposta i permessi del file .env in modo che sia leggibile solo dal tuo utente, ad esempio:
chmod 600 ./duckdns-updater/.env

🏗️ Build delle immagini e avvio dei container
Per buildare le immagini (se definite tramite build: con URL GitHub) e avviare tutto in background:
docker compose up -d --build
🧱 Il flag --build forza Docker a scaricare i Dockerfile remoti ed eseguire la build, anche se l'immagine esiste già localmente.
🔍 Verifica che tutto sia partito correttamente
docker compose ps
Puoi anche consultare i log con:
docker compose logs -f
🔁 Aggiornare il repository e ricostruire tutto (quando aggiorni da GitHub)
git pull
docker compose down
docker compose up -d --build

Parte 4: Il Vigile Urbano del Traffico (Nginx Proxy Manager) 🚦
4.1: Accendiamo i Motori! 🔥
Ora che tutto è pronto, avviamo il nostro stack.
Bash
docker compose up -d --build
Sii paziente, il primo avvio richiederà tempo per costruire tutte le immagini.
4.2: Il Certificato Magico di Tailscale 📜
1.	Accedi a NPM: Vai su http://ip_tailscale:8181 (Login: admin@example.com / changeme). Cambia subito le credenziali!
2.	Ottieni il certificato dal terminale del server:
Bash
# Sostituisci con il nome completo del tuo server
tailscale cert nome-server.tua-tailnet.ts.net 
o	Nota per macOS 🍏: Se il comando non stampa il certificato ma dice di aver scritto dei file, usa questi comandi per visualizzarli:
Bash
cat "$HOME/Library/Containers/io.tailscale.ipn.macos/Data/nome-server.tua-tailnet.ts.net.key"
cat "$HOME/Library/Containers/io.tailscale.ipn.macos/Data/nome-server.tua-tailnet.ts.net.crt"
3.	Carica il certificato su NPM:
o	In NPM, vai su SSL Certificates > Add SSL Certificate > scheda Custom.
o	Crea due file sul tuo PC (privato.key e pubblico.crt), incolla dentro il contenuto ottenuto dal terminale e caricali nei rispettivi campi.
o	Dai un nome facile da ricordare (es. "Certificato Tailscale") e salva.
4.3: Smistiamo il Traffico (Proxy & Stream Hosts)
Useremo un Proxy Host per il servizio principale e degli Stream per gli altri.
•	Configura il Proxy Host per Mammamia (sulla porta 443 standard):
1.	Vai su Hosts -> Proxy Hosts -> Add Proxy Host.
2.	Domain Names: nome-server.tua-tailnet.ts.net.
3.	Forward Hostname / IP: mammamia.
4.	Forward Port: 8080.
5.	Scheda SSL: Seleziona "Certificato Tailscale" e attiva Force SSL.
6.	Salva.
•	Configura gli Stream per gli altri servizi (sulle porte personalizzate):
1.	Vai su Hosts -> Streams -> Add Stream.
2.	Per AIOStreams:
	Incoming Port: 8001
	Forward Hostname / IP: aiostreams
	Forward Port: 3000
3.	Per StreamV:
	Incoming Port: 8002
	Forward Hostname / IP: streamv
	Forward Port: 7860
4.	Per MediaFlow Proxy (mfp):
•	Incoming Port: 8003
•	Forward Hostname / IP: mfp
•	Forward Port: 8888
 
Parte 5: La Configurazione Finale di Stremio 🎬
Ci siamo quasi!
1.	Attiva Tailscale sul dispositivo da cui vuoi usare Stremio.
2.	Apri Stremio, vai su Addons (icona 🧩) > I Miei Addon e disinstalla le vecchie versioni.
3.	Reinstalla gli addon usando i nuovi indirizzi sicuri:
o	https://nome-server.tua-tailnet.ts.net (per Mammamia)
o	https://nome-server.tua-tailnet.ts.net:8001 (per AIOStreams)
o	https://nome-server.tua-tailnet.ts.net:8002 (per StreamV)
o	https://nome-server.tua-tailnet.ts.net:8003 (per MFP)
Certamente! Ottima osservazione, completiamo la guida aggiungendo i passaggi specifici per MediaFlow Proxy (mfp) nella sezione 4.3, così da avere un documento davvero completo.
Ecco la versione finale e corretta della guida.
 
Guida Definitiva: Il Tuo Stremio Stack Self-Hosted 🚀
Sicuro con Docker 🐳, Privato con Tailscale 🛡️, Accessibile Ovunque 🌍
Ciao a tutti, amici nerd! 👋
Siete stanchi di configurazioni complicate e volete il pieno controllo del vostro streaming? Perfetto! Questa guida vi accompagnerà passo passo nella creazione di un'istanza Stremio-addons privata, usando la magia di Docker e la sicurezza di Tailscale.
L'obiettivo finale? Avere il vostro "Netflix" personale, accessibile in modo sicuro dal vostro telefono in 4G o dal PC di un amico, senza mai esporre la vostra rete di casa su Internet.
Basta chiacchiere, rimbocchiamoci le maniche!
 
Indice dei Contenuti 🗺️
1.	Cosa Ti Serve Prima di Iniziare? 🛠️
2.	Parte 1: Installiamo le Basi (Docker & Git)
3.	Parte 2: Creiamo la Nostra Rete Privata con Tailscale
4.	Parte 3: Diamo una Casa ai Nostri Servizi (Il Progetto Docker)
o	3.1: La Cartella Principale e la Rete "Ponte"
o	3.2: Il Cuore del Progetto: docker-compose.yml
o	3.3: I File Segreti: Creazione dei .env
5.	Parte 4: Il Vigile Urbano del Traffico (Nginx Proxy Manager)
o	4.1: Accendiamo i Motori!
o	4.2: Il Certificato Magico di Tailscale 📜
o	4.3: Smistiamo il Traffico (Proxy & Stream Hosts)
6.	Parte 5: La Configurazione Finale di Stremio 🎬
7.	Parte 6: Mantenere la Barca a Galla (Aggiornamenti)
8.	SOS? Sezione Troubleshooting 🆘
 
Cosa Ti Serve Prima di Iniziare? 🛠️
•	Un Serverino 🖥️: Un PC, un Mac Mini, un vecchio laptop, un Raspberry Pi 4 (o superiore), o un NAS che possa rimanere sempre acceso.
•	Sistema Operativo: Consigliato un Linux basato su Debian (es. Ubuntu Server 22.04 LTS 🐧). Se usi macOS 🍏, i passaggi sono identici una volta installato Docker Desktop.
•	Account Gratuiti:
o	Un account GitHub 🐙
o	Un account Tailscale ✨
 
Parte 1: Installiamo le Basi (Docker & Git)
Questi comandi sono per Ubuntu/Debian.
1.1: Installazione di Git 🔧
Bash
sudo apt update
sudo apt install git -y
1.2: Installazione di Docker e Docker Compose 🐳
Installiamo la versione più recente e ufficiale.
Bash
# Facciamo pulizia di eventuali vecchie versioni
sudo apt remove docker docker-engine docker.io containerd runc -y

# Installiamo i pacchetti necessari per la comunicazione sicura
sudo apt install ca-certificates curl gnupg lsb-release -y

# Aggiungiamo la chiave GPG ufficiale di Docker per la verifica
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Aggiungiamo il repository ufficiale di Docker
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Finalmente, installiamo tutto il pacchetto Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Aggiungiamo il nostro utente al gruppo 'docker' per non dover scrivere 'sudo' ogni volta
sudo usermod -aG docker $USER

# Applichiamo le modifiche (potrebbe essere necessario fare logout e login)
newgrp docker
Verifica che tutto sia a posto con docker --version. Dovrebbe mostrarti la versione installata.
 
Parte 2: Creiamo la Nostra Rete Privata con Tailscale 🛡️
1.	Installa Tailscale Ovunque: Vai sul sito di Tailscale e installa l'app sul tuo server e su tutti i dispositivi da cui vorrai usare Stremio (telefono, PC, tablet, etc.).
2.	Accedi con lo Stesso Account: È fondamentale usare lo stesso account di login su tutti i dispositivi per farli entrare nella stessa rete.
3.	Abilita la Magia ✨: Vai alla pagina DNS della tua console Tailscale e assicurati che queste due opzioni siano abilitate:
o	MagicDNS
o	HTTPS Certificates
4.	Trova il Nome del Tuo Server: Nella pagina Machines, trova il tuo server e copia il suo nome macchina completo (es. miniserver.taila0b363.ts.net). Tienilo a portata di mano!
 
Parte 3: Diamo una Casa ai Nostri Servizi (Il Progetto Docker) 🏠
3.1: La Cartella Principale e la Rete "Ponte"
Sul server, creiamo la cartella che conterrà tutte le nostre configurazioni.
Bash
cd ~
mkdir stremio-stack
cd stremio-stack
Ora creiamo la rete "ponte" che Nginx userà per comunicare con gli altri servizi.
Bash
docker network create proxy
3.2: Il Cuore del Progetto: docker-compose.yml 📄
Questo file è la ricetta per il nostro intero stack. Crealo con nano docker-compose.yml e incollaci questo contenuto.
YAML
# --- Inizio del file docker-compose.yml ---
services:
  # Nginx Proxy Manager (NPM): il nostro vigile urbano 👮 che smista il traffico
  npm:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: npm
    restart: unless-stopped
    ports:
      - '80:80'    # Porta HTTP
      - '443:443'  # Porta HTTPS
      - '8181:81'  # Porta per l'interfaccia di amministrazione
      # Porte personalizzate per gli altri servizi
      - '8001:8001' # Porta per AIOStreams
      - '8002:8002' # Porta per StreamV
      - '8003:8003' # Porta per MediaFlow Proxy
    volumes:
      - ./npm/data:/data
      - ./npm/letsencrypt:/etc/letsencrypt
    networks:
      - proxy

  # Mammamia: il nostro addon principale 🍿
  mammamia:
    build: https://github.com/openserials/mammamia-plat.git
    container_name: mammamia
    restart: unless-stopped
    networks:
      - proxy
    env_file:
      - ./mammamia/.env

  # AIOStreams: un altro ottimo addon 🚀
  aiostreams:
    image: ghcr.io/viren070/aiostreams:latest
    container_name: aiostreams
    restart: unless-stopped
    networks:
      - proxy
    env_file:
      - ./aiostreams/.env
        
  # StreamV: addon per lo streaming 📺
  streamv:
    build: https://github.com/PX-C/streamv.git
    container_name: streamv
    restart: unless-stopped
    networks:
      - proxy

  # MediaFlow Proxy: un proxy per alcuni flussi (opzionale) 🔀
  mediaflow_proxy:
    build: https://github.com/MH-Med-Bot/mediaflow-proxy-ita.git
    container_name: mfp
    restart: unless-stopped
    networks:
      - proxy
    env_file:
      - ./mfp/.env

# Definizione della rete esterna che collega tutto
networks:
  proxy:
    external: true
# --- Fine del file docker-compose.yml ---
3.3: I File Segreti: Creazione dei .env 🤫
Questi file contengono le chiavi e le impostazioni personali per ogni servizio.
Per Mammamia:
Bash
mkdir mammamia
touch mammamia/.env
nano mammamia/.env
Incolla la tua chiave API di The Movie Database (TMDB), che puoi ottenere gratuitamente dal loro sito.
Snippet di codice
TMDB_KEY=LA_TUA_CHIAVE_API_TMDB_QUI
Per AIOStreams:
Bash
mkdir aiostreams
touch aiostreams/.env
nano aiostreams/.env
Incolla le configurazioni minime. Puoi generare una SECRET_KEY sicura con il comando openssl rand -hex 32.
Snippet di codice
# Sostituisci <nome-server.tua-tailnet.ts.net> con il tuo nome server Tailscale
BASE_URL=https://<nome-server.tua-tailnet.ts.net>:8001
ADDON_ID=community.aiostreams.selfhosted
SECRET_KEY=LA_TUA_CHIAVE_SEGRETA_GENERATA_CON_OPENSSL
Per MediaFlow Proxy:
Bash
mkdir mfp
touch mfp/.env
nano mfp/.env
Un esempio di configurazione base:
Snippet di codice
API_PASSWORD=una_password_molto_segreta
FORWARDED_ALLOW_IPS=*
 
Parte 4: Il Vigile Urbano del Traffico (Nginx Proxy Manager) 🚦
4.1: Accendiamo i Motori! 🔥
Ora che tutto è pronto, avviamo il nostro stack.
Bash
docker compose up -d --build
Sii paziente, il primo avvio richiederà tempo per costruire tutte le immagini.
4.2: Accesso e Setup Iniziale di NPM
•	Vai su http://IP_LOCALE_DEL_SERVER:8181.
•	Login: admin@example.com / changeme. Cambia subito le credenziali!
4.3: Ottenere e Caricare il Certificato SSL di Tailscale 📜
1.	Ottieni il certificato dal terminale del server:
Bash
# Sostituisci con il nome completo del tuo server
tailscale cert nome-server.tua-tailnet.ts.net 
o	Nota per macOS 🍏: Se il comando non stampa il certificato ma dice di aver scritto dei file, usa questi comandi per visualizzarli:
Bash
cat "$HOME/Library/Containers/io.tailscale.ipn.macos/Data/nome-server.tua-tailnet.ts.net.key"
cat "$HOME/Library/Containers/io.tailscale.ipn.macos/Data/nome-server.tua-tailnet.ts.net.crt"
2.	Carica il certificato su NPM:
o	In NPM, vai su SSL Certificates > Add SSL Certificate > scheda Custom.
o	Crea due file sul tuo PC (privato.key e pubblico.crt), incolla dentro il contenuto ottenuto dal terminale e caricali nei rispettivi campi.
o	Dai un nome facile da ricordare (es. "Certificato Tailscale") e salva.
4.4: Smistiamo il Traffico (Proxy & Stream Hosts)
Useremo un Proxy Host per il servizio principale (sulla porta 443 standard) e degli Stream per tutti gli altri servizi su porte personalizzate.
•	Configura il Proxy Host per Mammamia (servizio principale):
1.	Vai su Hosts -> Proxy Hosts -> Add Proxy Host.
2.	Domain Names: nome-server.tua-tailnet.ts.net.
3.	Forward Hostname / IP: mammamia.
4.	Forward Port: 8080.
5.	Scheda SSL: Seleziona "Certificato Tailscale" e attiva Force SSL.
6.	Salva.
•	Configura gli Stream per gli altri servizi:
1.	Vai su Hosts -> Streams -> Add Stream.
2.	Per AIOStreams:
	Incoming Port: 8001
	Forward Hostname / IP: aiostreams
	Forward Port: 3000
3.	Per StreamV:
	Incoming Port: 8002
	Forward Hostname / IP: streamv
	Forward Port: 7860
4.	Per MediaFlow Proxy (mfp):
	Incoming Port: 8003
	Forward Hostname / IP: mfp
	Forward Port: 8888
5.	(Aggiungi altri stream se necessario...)
 
Parte 5: La Configurazione Finale di Stremio 🎬
Ci siamo quasi!
1.	Attiva Tailscale sul dispositivo da cui vuoi usare Stremio.
2.	Apri Stremio, vai su Addons (icona 🧩) > I Miei Addon e disinstalla le vecchie versioni.
3.	Reinstalla gli addon usando i nuovi indirizzi sicuri:
o	https://nome-server.tua-tailnet.ts.net (per Mammamia)
o	https://nome-server.tua-tailnet.ts.net:8001 (per AIOStreams)
o	https://nome-server.tua-tailnet.ts.net:8002 (per StreamV)
o	https://nome-server.tua-tailnet.ts.net:8003 (per MediaFlow Proxy)
 
Parte 6: Mantenere la Barca a Galla (Aggiornamenti) 🔄
•	Per aggiornare un servizio da GitHub (es. streamv):
Bash
docker compose build --pull --no-cache streamv
docker compose up -d
•	Per aggiornare un servizio da un'immagine Docker (es. aiostreams):
Bash
docker compose pull aiostreams
docker compose up -d
 
SOS? Sezione Troubleshooting 🆘
•	L'indirizzo .ts.net non funziona?
o	Tailscale è attivo sul dispositivo che stai usando?
o	MagicDNS è abilitato nella console di Tailscale?
•	Vedo la vecchia grafica dopo un aggiornamento?
o	Fai un hard-refresh del browser (su Mac: Cmd+Shift+R, su Windows: Ctrl+F5).
•	Un container è in errore?
o	Controlla i suoi log: docker compose logs nome_servizio.
o	Utilizza servizi di IA come Google Gemini o ChatGPT per risolvere i problemi. Il trucco è copiare e incollare gli errori del terminale
 
Conclusione 🎉
Congratulazioni! Hai appena costruito un media center personale, privato, sicuro e incredibilmente potente. Ora sei tu il padrone del tuo streaming.


