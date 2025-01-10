---
layout: single
title:  "SSH Cheat-Sheet"
date:   2025-01-04 12:00:06 +0100
categories: Blog SSH Linux
comments: true
author: Tom
---

Häufig genutzte Kommandos und Config in Verbdinndung mit der SecureShell (=SSH)

# Wichtige Verzeichnisse & Dateien

```ruby

~/.ssh/                 # Ablage der SSH Keys
~/.ssh/authorized_keys  # Liste der öffentlichen Keys mit deren privatem Pendant man Zugriff sich als Nutzer auf dem Server authentifizieren kann
~/.ssh/known_hosts      # Liste der SSH Server Public Keys von remote Hosts. Bei jedem Connect zu einem Server wird überprüft ob sich der Schlüssel geändert hat
~/.ssh/config           # SSH User Config (z.B. welcher User, Schlüssel etc. für welchen Host genutzt wird)


```

# Kommandos

```ruby
# Startet den SSH Agent und erzeugt die notwendigen Umgebungsvariablen
eval ($ssh-agent)       

# Kopiert den Public Key in die Files ~/.ssh/authorizes_keys auf einem remote Server
ssh-copy-id  -i </pathto/ssh_public_key.pub> username@server.domain # 

# Neues Schlüsselpaar erstellen
ssh-keygen -t ed25519 -f ~/.ssh/ssh_<Servername>_<Filename>

# Host Eintrag aus der ~/.ssh/known_hosts
ssh-keygen -f  "~/.ssh/known_hosts" -R "server.domain.tld"

```
