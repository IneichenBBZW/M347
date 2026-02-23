# WSL 2 mit Ubuntu installieren

## Ziel

Du installierst  **Ubuntu unter Windows**.
Ubuntu läuft **in WSL 2** (Windows Subsystem for Linux).

Damit kannst du:

* Linux lernen
* Python programmieren
* Docker verwenden

## Voraussetzungen

* Windows 10 oder Windows 11
* Internetverbindung
* Administrator-Rechte

## 1️⃣ Windows Terminal oder PowerShell öffnen

1. Klicke auf **Start**
2. Tippe **PowerShell**
3. Rechtsklick → **Als Administrator ausführen**

## 2️⃣ WSL aktivieren

Gib folgenden Befehl ein:

```powershell
wsl --install
```

👉 Dieser Befehl:

* aktiviert WSL
* installiert WSL 2
* installiert **Ubuntu** als Standard

## 3️⃣ Computer neu starten

Nach der Installation:

👉 **Windows neu starten**

## 4️⃣ Ubuntu starten

Nach dem Neustart:

1. Öffne das **Startmenü**
2. Suche nach **Ubuntu**
3. Starte **Ubuntu**

Beim ersten Start:

* Benutzername wählen (frei)
* Passwort setzen (wird nicht angezeigt)

## 5️⃣ Prüfen: Läuft WSL 2?

Öffne **PowerShell** (normal, nicht Admin):

```powershell
wsl -l -v
```

Beispielausgabe:

```
NAME      STATE   VERSION
Ubuntu    Running 2
```

✔️ **VERSION 2** = korrekt
❌ VERSION 1 → falsch

## 6️⃣ Ubuntu auf WSL 2 umstellen (falls nötig)

Falls dort **VERSION 1** steht:

```powershell
wsl --set-version Ubuntu 2
```

Danach nochmals prüfen:

```powershell
wsl -l -v
```

## 7️⃣ Ubuntu aktualisieren

In **Ubuntu** (Linux-Fenster):

```bash
sudo apt update
sudo apt upgrade -y
```

## 8️⃣ Ubuntu-Version prüfen

```bash
lsb_release -a
```

Beispiel:

```
Ubuntu 24.04 LTS
```

## ❓ Wie wählt man Ubuntu für WSL 2 aus?

### Möglichkeit A (einfach)

```powershell
wsl --install
```

👉 installiert **Ubuntu automatisch**

### Möglichkeit B (gezielt auswählen)

1. Verfügbare Distributionen anzeigen:

```powershell
wsl --list --online
```

2. Ubuntu installieren:

```powershell
wsl --install -d Ubuntu
```

Oder z. B.:

```powershell
wsl --install -d Ubuntu-24.04
```

# VS Code für WSL 2 einrichten

## Ziel

Du richtest **Visual Studio Code** so ein,
dass es **direkt mit Ubuntu in WSL 2** arbeitet.

Damit kannst du:

* Linux-Dateien bearbeiten
* Python programmieren
* später Docker nutzen

## Voraussetzungen

* Windows 10 oder Windows 11
* WSL 2 ist installiert
* Ubuntu läuft in WSL 2
* Internetverbindung

## 1️⃣ Visual Studio Code installieren (Windows)

1. Öffne den Browser
2. Gehe auf: [https://code.visualstudio.com](https://code.visualstudio.com/)
3. Lade **Visual Studio Code** herunter
4. Starte die Installation
5. Übernimm die Standard-Einstellungen

👉 VS Code läuft jetzt  **unter Windows**.

**Schulalltag-Beispiel:**
VS Code ist dein Heft. WSL ist der Schulserver.

## 2️⃣ Erweiterung **WSL** installieren

1. Starte **VS Code**
2. Öffne den **Extensions-Bereich**
3. Suche nach **WSL**
4. Wähle **WSL (Microsoft)**
5. Klicke **Installieren**

👉 VS Code kann jetzt mit **Ubuntu in WSL 2** arbeiten.

## 3️⃣ VS Code mit Ubuntu verbinden

1. Öffne **Ubuntu** (WSL)
2. Tippe im Terminal:

```bash
code .
```

3. VS Code öffnet sich automatisch
4. Du arbeitest jetzt **in Linux**

Unten links siehst du:

```
WSL: Ubuntu
```

✔️ Verbindung aktiv

**Schulalltag-Beispiel:**
Du arbeitest im Klassenzimmer,
aber deine Dateien liegen im Lehrer-PC.

## 4️⃣ Erweiterung **Python (Microsoft)** installieren

⚠️ Wichtig:
Diese Erweiterung wird  **in WSL installiert**, nicht nur in Windows.

1. In VS Code: **Extensions öffnen**
2. Suche nach **Python**
3. Wähle **Python (Microsoft)**
4. Klicke **Installieren**

👉 VS Code erkennt jetzt:

* Python-Version
* virtuelle Umgebungen
* Fehler im Code

## 5️⃣ Test: Python in WSL verwenden

1. Öffne in VS Code ein neues Terminal
2. Prüfe Python:

```bash
python3 --version
```

Beispiel:

```
Python 3.12.x
```

✔️ Python läuft korrekt

# Docker installieren (WSL 2 + Ubuntu)

## Ziel

Du installierst **Docker** unter  **Ubuntu in WSL 2**.
Danach kannst du:

* Container starten
* Programme isoliert ausführen
* mit Python und Docker arbeiten

## Voraussetzungen

* Windows 10 oder Windows 11
* WSL 2 ist installiert
* Ubuntu läuft in WSL 2
* Internetverbindung

## 1️⃣ Ubuntu starten

1. Öffne das **Startmenü**
2. Starte **Ubuntu**
3. Du siehst ein **Linux-Terminal**

## 2️⃣ System aktualisieren

```bash
sudo apt update
sudo apt upgrade -y
```

👉 Holt aktuelle Pakete.

## 3️⃣ Benötigte Werkzeuge installieren

```bash
sudo apt install -y \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
```

👉 Diese Pakete braucht Docker.

## 4️⃣ Docker-Schlüssel hinzufügen

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Rechte setzen:

```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

## 5️⃣ Docker-Paketquelle hinzufügen

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" \
| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## 6️⃣ Docker installieren

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

👉 Installiert:

* Docker Engine
* Docker CLI
* Docker Compose (v2)

## 7️⃣ Docker ohne `sudo` nutzen

```bash
sudo usermod -aG docker $USER
```

⚠️ Danach  **WSL neu starten**:

### In PowerShell:

```powershell
wsl --shutdown
```

Ubuntu danach wieder starten.

## 8️⃣ Installation testen

```bash
docker version
```

Test-Container starten:

```bash
docker run hello-world
```

👉 Wenn Text erscheint: **Docker läuft korrekt** ✅

## 9️⃣ Docker Compose testen

```bash
docker compose version
```

# Docker mit Python installieren (WSL 2 + Ubuntu)

## Ziel

Du installierst:

* **Docker** (Container-System)
* **Python**
* **Docker für Python** (Python steuert Docker)

Am Ende kannst du:

* Container starten
* Docker mit Python bedienen

## Voraussetzungen

* Windows 10 oder Windows 11
* WSL 2 ist installiert
* Ubuntu läuft in WSL 2
* Docker ist installiert
  (siehe Anleitung  *Docker installieren (WSL 2 + Ubuntu)*)

## 1️⃣ Ubuntu starten

1. Startmenü öffnen
2. **Ubuntu** starten
3. Terminal erscheint

## 2️⃣ Prüfen: Docker läuft

```bash
docker version
```

Wenn eine Versionsnummer erscheint → **Docker läuft** ✅

## 3️⃣ Python-Version prüfen

```bash
python3 --version
```

Beispiel:

```
Python 3.12.x
```

## 4️⃣ Virtuelle Python-Umgebung erstellen

Wir arbeiten **nicht** direkt im System.

```bash
python3 -m venv venv
```

## 5️⃣ Virtuelle Umgebung aktivieren

```bash
source venv/bin/activate
```

Du siehst jetzt:

```
(venv)
```

## 6️⃣ pip aktualisieren (nur im venv)

```bash
python -m pip install --upgrade pip
```

## 7️⃣ Docker für Python installieren

```bash
pip install docker
```

👉 Das ist **nicht** Docker selbst.
👉 Das ist das  **Python-Werkzeug**, um Docker zu steuern.

## 8️⃣ Test: Docker mit Python ansprechen

### Python starten

```bash
python
```

### Code eingeben

```python
import docker

client = docker.from_env()
print(client.version())
```

Wenn Infos angezeigt werden → **alles korrekt** ✅

## 9️⃣ Beispiel: Container mit Python starten

```python
container = client.containers.run(
    "hello-world",
    detach=True
)
print(container.id)
```

👉 Python startet einen Docker-Container.
