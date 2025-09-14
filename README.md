# selfhost
Stremio stack with StreamVix, Mammamia, AIOtreams and MFP self-hosted using Tailscale

Ciao a tutti!

Questa guida vi mostrerà come creare da zero un'infrastruttura Stremio privata e self-hosted. L'obiettivo è avere i propri addon (come Mammamia, AIOStreams, etc.) in esecuzione su un server casalingo, accessibili in modo totalmente sicuro da qualsiasi dispositivo (PC, telefono, TV) e da qualsiasi luogo, senza aprire alcuna porta sul router e senza esporre il proprio IP pubblico.

Software e Prerequisiti

Hardware: Un computer che rimarrà sempre acceso. Può essere un mini-PC, un vecchio laptop, un Raspberry Pi 4 (o superiore), o un NAS.

Sistema Operativo: Una distribuzione Linux basata su Debian (come Ubuntu Server 22.04 LTS) è ideale. La guida è basata su questa, ma i comandi Docker sono universali. Su macOS, i passaggi sono identici una volta installato Docker Desktop.

Account Gratuiti:

Un account GitHub (per accedere ad alcuni repository).

Un account Tailscale.



