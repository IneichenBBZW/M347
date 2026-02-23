# Erste Schritte mit Docker – gesteuert mit Python

## Was du hier lernst

* Du steuerst Docker **mit Python**
* Du startest **Container**
* Du siehst  **klare Ausgaben**, die du verstehen kannst

👉 Wichtig:
Docker  **muss bereits installiert und gestartet sein**.
Python steuert Docker nur – es installiert Docker nicht selbst.

## Vorbereitung (einmal nötig)

### 1. Docker starten

* Windows / Mac: **Docker Desktop starten**
* Linux: Docker-Dienst starten (z. B. über das System)

👉 Docker muss laufen, sonst funktioniert nichts.

### 2. Python-Bibliothek installieren

Im Terminal:

```bash
pip install docker
```

👉 Damit kann Python mit Docker sprechen.

## Python starten

### Variante A: Python-REPL (direkt ausprobieren)

```bash
python
```

Du siehst z. B.:

```
Python 3.x.x ...
>>>
```

### Variante B: Python-Datei

Du kannst später auch eine `.py`-Datei verwenden.

## Verbindung zu Docker prüfen

Gib in Python ein:

```python
import docker
client = docker.from_env()
client.ping()
```

### Ausgabe:

```
True
```

### Bedeutung:

✅ Python erreicht Docker
✅ Docker läuft
✅ Alles bereit

## Infos über Docker anzeigen

```python
info = client.version()

print("Docker-Version:", info.get("Version"))
print("API-Version:", info.get("ApiVersion"))
print("Betriebssystem:", info.get("Os"))
print("Architektur:", info.get("Arch"))
```

### Beispiel-Ausgabe:

```
Docker-Version: 25.0.3
API-Version: 1.44
Betriebssystem: linux
Architektur: amd64
```

👉 Docker ist ein  **Server**, Python spricht mit ihm über eine  **API**.

## Erstes Image herunterladen

Ein **Image** ist eine Vorlage für Container.

```python
client.images.pull("ubuntu:latest")
print("Ubuntu-Image ist bereit.")
```

### Ausgabe:

```
Ubuntu-Image ist bereit.
```

## Ersten Container starten

```python
output = client.containers.run(
    "ubuntu:latest",
    "echo Hallo aus dem Container!",
    remove=True
)

print(output.decode())
```

### Ausgabe:

```
Hallo aus dem Container!
```

### Was ist passiert?

* Docker startet einen Container
* Der Container führt einen Befehl aus
* Danach wird er automatisch gelöscht

## Anderes Image testen (Alpine)

```python
client.images.pull("alpine:latest")

output = client.containers.run(
    "alpine:latest",
    "echo Hallo von Alpine",
    remove=True
)

print(output.decode())
```

### Ausgabe:

```
Hallo von Alpine
```

👉 Verschiedene Linux-Systeme – gleicher Ablauf.

## Container im Hintergrund starten

```python
container = client.containers.run(
    "alpine:latest",
    "sleep infinity",
    detach=True
)

print("Container gestartet:", container.name)
```

### Beispiel:

```
Container gestartet: friendly_pike
```

## Laufende Container anzeigen

```python
for c in client.containers.list():
    print(c.name, "-", c.status)
```

### Beispiel-Ausgabe:

```
friendly_pike - running
```

## IP-Adresse des Containers anzeigen

```python
container.reload()
networks = container.attrs["NetworkSettings"]["Networks"]

for name, data in networks.items():
    print("Netzwerk:", name)
    print("IP-Adresse:", data["IPAddress"])
```

### Beispiel:

```
Netzwerk: bridge
IP-Adresse: 172.17.0.2
```

👉 Jeder Container bekommt  **seine eigene IP-Adresse**.

## Container stoppen und löschen

```python
container.stop()
container.remove()
print("Container beendet und gelöscht.")
```

### Ausgabe:

```
Container beendet und gelöscht.
```

## Alles in einem Python-Skript

Datei: `docker_start.py`

```python
import docker

client = docker.from_env()

print("Verbindung:", client.ping())

print("\nDocker-Info:")
info = client.version()
print(info.get("Version"), "-", info.get("Os"))

print("\nUbuntu-Test:")
out = client.containers.run(
    "ubuntu:latest",
    "echo Hallo aus Python und Docker!",
    remove=True
)
print(out.decode())

print("\nFertig.")
```

Starten mit:

```bash
python docker_start.py
```

## Alle Container löschen

Datei: `container_loeschen.py`

```python
import docker

client = docker.from_env()

for container in client.containers.list(all=True):
    print("Stoppe:", container.name)
    container.stop()

    print("Lösche:", container.name)
    container.remove()
```

Starten mit:

```bash
python container_loeschen.py
```

## Alle Images löschen

Datei: `images_loeschen.py`

```python
import docker

client = docker.from_env()

for image in client.images.list():
    try:
        print("Lösche Image:", image.short_id)
        client.images.remove(image.id)
    except docker.errors.APIError as e:
        print("Übersprungen:", image.short_id)

```

Starten mit:

```bash
python images_loeschen.py
```


## Verbindung zu Docker prüfen

### Python

```python
client.ping()
```

### Docker-CLI

```bash
docker info
```

👉 Wenn Infos erscheinen, läuft Docker.

## Docker-Version und Systeminfos

### Python

```python
info = client.version()
```

### Docker-CLI

```bash
docker version
```

oder ausführlicher:

```bash
docker info
```

## Image herunterladen (pull)

### Python

```python
client.images.pull("ubuntu:latest")
```

### Docker-CLI

```bash
docker pull ubuntu:latest
```

## Container starten und Befehl ausführen (einmalig)

### Python

```python
client.containers.run(
    "ubuntu:latest",
    "echo Hallo aus dem Container!",
    remove=True
)
```

### Docker-CLI

```bash
docker run --rm ubuntu:latest echo "Hallo aus dem Container!"
```

**Erklärung:**

* `--rm` → Container wird nach dem Lauf gelöscht
* `echo ...` → Befehl im Container

## Anderes Image testen (Alpine)

### Python

```python
client.containers.run(
    "alpine:latest",
    "echo Hallo von Alpine",
    remove=True
)
```

### Docker-CLI

```bash
docker run --rm alpine:latest echo "Hallo von Alpine"
```

## Container im Hintergrund starten

### Python

```python
container = client.containers.run(
    "alpine:latest",
    "sleep infinity",
    detach=True
)
```

### Docker-CLI

```bash
docker run -d alpine:latest sleep infinity
```

**Erklärung:**

* `-d` → detached (läuft im Hintergrund)

## Laufende Container anzeigen

### Python

```python
client.containers.list()
```

### Docker-CLI

```bash
docker ps
```

Alle Container (auch gestoppte):

```bash
docker ps -a
```

## Container-Name anzeigen

### Python

```python
container.name
```

### Docker-CLI

```bash
docker ps
```

(Spalte  **NAMES** )

## IP-Adresse eines Containers anzeigen

### Python

```python
container.attrs["NetworkSettings"]["Networks"]
```

### Docker-CLI

```bash
docker inspect <container-name>
```

Nur IP-Adresse:

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container-name>
```

## Container stoppen

### Python

```python
container.stop()
```

### Docker-CLI

```bash
docker stop <container-name>
```

## Container löschen

### Python

```python
container.remove()
```

### Docker-CLI

```bash
docker rm <container-name>
```

Stoppen **und** löschen:

```bash
docker rm -f <container-name>
```

## Alle Container stoppen

### Python

```python
for c in client.containers.list(all=True):
    c.stop()
```

### Docker-CLI

```bash
docker stop $(docker ps -aq)
```

## Alle Container löschen

### Python

```python
for c in client.containers.list(all=True):
    c.remove()
```

### Docker-CLI

```bash
docker rm $(docker ps -aq)
```

## Alle Images anzeigen

### Python

```python
client.images.list()
```

### Docker-CLI

```bash
docker images
```

## Image löschen

### Python

```python
client.images.remove(image.id)
```

### Docker-CLI

```bash
docker rmi <image-id>
```

## Alle Images löschen

### Python

```python
for image in client.images.list():
    client.images.remove(image.id)
```

### Docker-CLI

```bash
docker rmi $(docker images -q)
```

## Merksatz

> **Python + Docker SDK = Fernbedienung**\
> **Docker CLI = direkt**

Beides steuert  **denselben Docker-Server**.
Nur die Sprache ist anders.
