# DL 4 Lernskript

> **Dockerfile-Grundstruktur** ,  **Layer-Prinzip** ,  **`.dockerignore`** ,  **Image bauen** ,  **Container starten** ,  **Ports veröffentlichen** ,  **Konfiguration über ENV** ,  **Build-Argumente/Labels** .

## Ablauf 90 Minuten

### 0–10 Einstieg: „Was ist im Image drin?“

* Ein Image ist die Grundlage für Container. Ein Container startet „aus einem Image“.
* Images bestehen aus mehreren Layern (Schichten). Beim Bauen erzeugt (fast) jede Dockerfile-Zeile einen neuen Layer.
* Mini-Demo (gleich im Lab): Wir lassen Python die Layer-History anzeigen (wie `docker history`).

### 10–30 Input/Demo: Dockerfile, Layer-Prinzip, `.dockerignore`

* Dockerfile = Rezept für ein Image. Es liegt meistens im Projekt-Root.
* Reihenfolge ist wichtig: Wenn du früh Code kopierst, wird Cache öfter ungültig → Builds dauern länger.
* `.dockerignore` verkleinert den „Build-Kontext“ und spart Zeit (z. B. `.git` nicht mitsenden).

### 30–75 Guided Lab: Python-Projekt → Dockerfile → Build → Run (Port)

* Lernende bauen ein eigenes Image Hello API
* Start als Container, Port wird veröffentlicht (`-p 8080:8080`-Prinzip)
* Konfiguration über ENV (WHO) wie im Buchbeispiel

### 75–90 Reflexion

* Was war schwierig? (Ports, Dependencies, „läuft der Container noch?“)
* Warum ist `.dockerignore` sinnvoll?
* Warum ist die Reihenfolge im Dockerfile wichtig?

## Vorbereitung für Lernende

### Checkliste

In Ubuntu/WSL2:

```bash
docker version
python3 --version
```

Für das Python-Docker-SDK: (im Projektordner)

```bash
python3 -m venv venv
source venv/bin/activate
pip install docker
```

## Guided Lab: „Hello API“ bauen (ohne CLI – mit Python-SDK)

### Schritt 1: Projektordner erstellen

```bash
mkdir hello-api
cd hello-api
```

### Schritt 2: `app.py` erstellen (kleine HTTP-API)

**Datei: `app.py`**

```python
import os
from http.server import BaseHTTPRequestHandler, HTTPServer


def get_who() -> str:
    return os.environ.get("WHO", "World")


def get_port() -> int:
    return int(os.environ.get("PORT", "8080"))


class HelloHandler(BaseHTTPRequestHandler):
    def do_GET(self) -> None:
        if self.path != "/":
            self.send_response(404)
            self.end_headers()
            self.wfile.write(b"Not Found\n")
            return

        message = f"Hallo {get_who()}. Ich wünsche, du wärst hier.\n"

        self.send_response(200)
        self.send_header("Content-Type", "text/plain; charset=utf-8")
        self.end_headers()
        self.wfile.write(message.encode("utf-8"))

    def log_message(self, format: str, *args) -> None:
        # Logs im Unterricht ruhig halten
        return


if __name__ == "__main__":
    server = HTTPServer(("0.0.0.0", get_port()), HelloHandler)
    server.serve_forever()
```

**Was macht die App?**

* Sie hat eine Route: `GET /`
* Sie liest `WHO` aus der Umgebung (Default: `"World"`)
  → genau das ist das Prinzip mit „Konfiguration über ENV“.

### Schritt 3: `requirements.txt` erstellen

**Datei: `requirements.txt`**

```txt
# keine externen Libraries nötig
```

*(Wir nutzen absichtlich keine externen Dependencies – Fokus ist Dockerfile/Images.)*

### Schritt 4: `.dockerignore` erstellen

**Datei: `.dockerignore`**

```txt
.git
venv
__pycache__
*.pyc
```

**Warum?**

* Docker schickt beim Build den Build-Kontext (Ordnerinhalt) an den Docker-Server.
* `.dockerignore` filtert unnötige Sachen raus (z. B. `.git`).

### Schritt 5: Dockerfile erstellen

**Datei: `Dockerfile`**

```dockerfile
FROM python:3.12-slim

# Build-Argument (nur beim Build verfügbar)
ARG email="student@example.com"

# Labels = Metadaten am Image
LABEL maintainer=$email
LABEL app="hello-api"

WORKDIR /app

# Erst "stabile" Dateien kopieren (Cache!)
COPY requirements.txt /app/

# Keine Pakete nötig, aber die Zeile zeigt das typische Muster:
RUN python -m pip install --upgrade pip

# Jetzt kommt der Code (ändert sich häufiger)
COPY app.py /app/app.py

ENV PORT=8080
EXPOSE 8080

CMD ["python", "app.py"]
```

**Wichtige Lernpunkte**

* Dockerfile beschreibt die Build-Schritte; jede Anweisung erzeugt typischerweise einen Layer.
* Reihenfolge: „Stabil & teuer“ zuerst, „Code“ eher spät → Cache wird besser genutzt.
* `ARG` ist für den Build; `ENV` wirkt zur Laufzeit (Konfiguration).
* `EXPOSE` ist „Dokumentation“: dieser Port ist gedacht. (Das eigentliche Port-Mapping passiert beim Start.)

## Schritt 6: Docker mit Python steuern (`import docker`)

**Datei: `docker_lab.py`**
*(Im gleichen Ordner wie Dockerfile)*

```python
# Author: Markus Ineichen
# Markus Ineichen, CC BY 4.0
import json
import time
from pathlib import Path
from urllib.request import urlopen

import fnmatch
import io
import tarfile
from pathlib import Path

import docker
from docker.errors import NotFound


PROJECT_DIR = Path(__file__).resolve().parent
IMAGE_TAG = "school/hello-api:latest"
CONTAINER_NAME = "hello-api"
HOST_PORT = 8080
CONTAINER_PORT = 8080

def docker_context_size_mb(project_dir: Path) -> float:
    # Patterns aus .dockerignore lesen
    ignore_file = project_dir / ".dockerignore"
    patterns: list[str] = []
    if ignore_file.exists():
        patterns = [
            line.strip()
            for line in ignore_file.read_text(encoding="utf-8").splitlines()
            if line.strip() and not line.strip().startswith("#")
        ]

    def is_ignored(rel_posix: str) -> bool:
        # Sehr einfache .dockerignore-Logik:
        # - fnmatch für Patterns wie "*.pyc"
        # - Prefix für Ordner wie "venv"
        for pat in patterns:
            pat = pat.rstrip("/")
            if pat and (fnmatch.fnmatch(rel_posix, pat) or rel_posix.startswith(pat + "/")):
                return True
        return False

    buf = io.BytesIO()
    with tarfile.open(fileobj=buf, mode="w") as tar:
        for p in project_dir.rglob("*"):
            if not p.is_file():
                continue
            rel = p.relative_to(project_dir).as_posix()
            if is_ignored(rel):
                continue
            tar.add(p, arcname=rel)

    return buf.tell() / 1024 / 1024

def remove_container_if_exists(client: docker.DockerClient, name: str) -> None:
    try:
        container = client.containers.get(name)
    except NotFound:
        return
    container.remove(force=True)

print(f"Build context size (dockerignore-aware): {docker_context_size_mb(PROJECT_DIR):.1f} MB")

def build_image(client: docker.DockerClient, maintainer_email: str) -> None:
    print(f"Building image '{IMAGE_TAG}' ...")
    start = time.perf_counter()

    # Low-level API: stabiler für Build-Logs
    stream = client.api.build(
        path=str(PROJECT_DIR),
        tag=IMAGE_TAG,
        buildargs={"email": maintainer_email},
        rm=True,
        decode=True,   # hier ist decode=True korrekt (liefert dicts)
    )

    for entry in stream:
        if "stream" in entry:
            print(entry["stream"], end="")
        elif "error" in entry:
            raise RuntimeError(entry["error"])

    image = client.images.get(IMAGE_TAG)

    elapsed = time.perf_counter() - start
    print(f"\nBuild finished in {elapsed:.1f}s, image id: {image.short_id}")

def show_image_labels(client: docker.DockerClient) -> None:
    image = client.images.get(IMAGE_TAG)
    labels = (image.attrs.get("Config", {}) or {}).get("Labels", {}) or {}
    print("Image labels:")
    print(json.dumps(labels, indent=2, ensure_ascii=False))


def show_image_layers(client: docker.DockerClient) -> None:
    # ähnlich wie "docker history"
    history = client.api.history(IMAGE_TAG)
    print("Top layers (history):")
    for i, item in enumerate(history[:8], start=1):
        size_mb = item.get("Size", 0) / 1024 / 1024
        created_by = (item.get("CreatedBy") or "").strip()
        print(f"{i:02d}. {size_mb:6.1f} MB  {created_by[:80]}")


def run_container(client: docker.DockerClient, who: str) -> None: # Im Schritt 8 hier ändern!
    print(f"Starting container '{CONTAINER_NAME}' ...")

    container = client.containers.run(
        IMAGE_TAG,
        name=CONTAINER_NAME,
        detach=True,
        ports={f"{CONTAINER_PORT}/tcp": HOST_PORT},     # wie "-p 8080:8080"
        environment={"WHO": who, "PORT": str(CONTAINER_PORT)},
    )

    # kurz warten, bis der Server lauscht
    time.sleep(1.0)

    url = f"http://127.0.0.1:{HOST_PORT}/"
    with urlopen(url, timeout=3) as resp:
        body = resp.read().decode("utf-8")

    print("API response:", body.strip())
    print("Container is running in background.")
    print("Stop/Remove later with: python docker_cleanup.py")


def main() -> None:
    client = docker.from_env()

    remove_container_if_exists(client, CONTAINER_NAME)
    build_image(client, maintainer_email="markus@example.com")

    show_image_labels(client)
    show_image_layers(client)

    run_container(client, who="World") # Im Schritt 8 hier ändern!


if __name__ == "__main__":
    main()
```

### Datei zum Aufräumen: `docker_cleanup.py`

```python
import docker
from docker.errors import NotFound

IMAGE_TAG = "school/hello-api:latest"
CONTAINER_NAME = "hello-api"


def main() -> None:
    client = docker.from_env()

    try:
        client.containers.get(CONTAINER_NAME).remove(force=True)
        print("Container removed.")
    except NotFound:
        print("Container not found.")

    # Image entfernen (optional)
    try:
        client.images.remove(IMAGE_TAG, force=True)
        print("Image removed.")
    except Exception as exc:
        print("Image not removed:", exc)


if __name__ == "__main__":
    main()
```

## Schritt 7: Ausführen

Im Projektordner:

```bash
source venv/bin/activate
python docker_lab.py
```

Dann testen (Ubuntu/WSL):

```bash
curl http://127.0.0.1:8080/
```

Oder im Windows-Browser:

* `http://localhost:8080/`

## Schritt 8: Konfiguration über ENV (WHO ändern)

Jetzt kommt das Prinzip „ENV als Konfiguration“: Container stoppen, neu starten mit anderer Variable.

1. Erst aufräumen:

```bash
python docker_cleanup.py
```

2. In `docker_lab.py` ändere:

```python
run_container(client, who="Sean and Karl")
```

3. Nochmal starten:

```bash
python docker_lab.py
```

Erwartung:

* Antwort ändert sich, ohne Codeänderung in `app.py`
  → Konfiguration über ENV.

## Mini-Zusammenfassung

* **Image** = Vorlage für Container.
* **Dockerfile** = Rezept, wie ein Image gebaut wird.
* **Layer** = Schichten im Image; Docker nutzt Cache, damit Builds schneller werden.
* **Build-Kontext** = Ordner, der an Docker geschickt wird (`.`). `.dockerignore` spart Zeit.
* **Port-Mapping** macht die API von aussen erreichbar (`8080:8080`).
* **ENV** ist Konfiguration zur Laufzeit (WHO).
* **ARG + LABEL** sind Metadaten/Build-Info (z. B. Maintainer-Mail).

## Reflexionsfragen (75–90)

1. Warum stoppt ein Container, wenn im Container kein Prozess läuft?
2. Was bringt `.dockerignore` im Unterrichtsalltag?
3. Was passiert beim Rebuild, wenn du nur **`app.py`** änderst? (Cache!)
4. Was war schwieriger: Ports oder ENV/Dependencies?

## Bonus-Aufgaben (siehe Übungen)

* **A:** Ändere `HOST_PORT` auf 8081 und teste im Browser.
* **B:** Bau das Image zweimal hintereinander und vergleiche Build-Zeiten (Cache).
* **C:** Füge ein Label `LABEL school_class="S-INA24x"` hinzu und gib es in Python aus.

## 1) Verbindung zu Docker (`docker.from_env()` / `ping()`)

### Python (implizit)

```python
client = docker.from_env()
client.ping()
```

### CLI

```bash
docker info
```

oder nur „läuft Docker?“:

```bash
docker version
```

## 2) Vorhandenen Container entfernen (`remove_container_if_exists(...).remove(force=True)`)

### Python

```python
container.remove(force=True)
```

### CLI

```bash
docker rm -f hello-api
```

(Wenn er nicht existiert, gibt’s eine Fehlermeldung – ist ok.)

## 3) Image bauen (`client.api.build(... tag=school/hello-api:latest, buildargs email=... )`)

### Python

```python
client.api.build(path=".", tag=IMAGE_TAG, buildargs={"email": "..."} ...)
```

### CLI

```bash
docker build -t school/hello-api:latest --build-arg email="markus@example.com" .
```

## 4) Labels am Image anzeigen (`show_image_labels()`)

### Python

```python
image.attrs["Config"]["Labels"]
```

### CLI (alle Labels)

```bash
docker inspect school/hello-api:latest --format '{{json .Config.Labels}}'
```

lesbarer (mit `jq`, falls installiert):

```bash
docker inspect school/hello-api:latest --format '{{json .Config.Labels}}' | jq
```

Oder „alles sehen“:

```bash
docker inspect school/hello-api:latest
```

## 5) Layer/History anzeigen (`client.api.history(IMAGE_TAG)`)

### Python

```python
client.api.history(IMAGE_TAG)
```

### CLI

```bash
docker history school/hello-api:latest
```

Optional ohne Truncation:

```bash
docker history --no-trunc school/hello-api:latest
```

## 6) Container starten: detach + Name + Port-Mapping + ENV (`containers.run(... detach=True, name=..., ports=..., environment=...)`)

### Python

```python
client.containers.run(
  IMAGE_TAG,
  name="hello-api",
  detach=True,
  ports={"8080/tcp": 8080},
  environment={"WHO": "World", "PORT": "8080"},
)
```

### CLI

```bash
docker run -d --name hello-api -p 8080:8080 -e WHO="World" -e PORT="8080" school/hello-api:latest
```

Wenn du später WHO änderst („Sean and Karl“):

```bash
docker rm -f hello-api
docker run -d --name hello-api -p 8080:8080 -e WHO="Sean and Karl" -e PORT="8080" school/hello-api:latest
```

## 7) Laufende Container anzeigen (wenn du es zeigen willst)

### Python (nicht im Script, aber typisch)

```python
client.containers.list()
```

### CLI

```bash
docker ps
```

## 8) Container-Logs (falls Lernende „läuft der noch?“ fragen)

### Python (würde gehen über `container.logs()`)

### CLI

```bash
docker logs hello-api
```

Live mitlaufen:

```bash
docker logs -f hello-api
```

## 9) HTTP-Test gegen die API (`urlopen(...)` / `curl ...`)

### Python

```python
urlopen("http://127.0.0.1:8080/")
```

### CLI

```bash
curl http://127.0.0.1:8080/
```

## 10) Stoppen/Remove Cleanup (`docker_cleanup.py`)

### Python

```python
client.containers.get("hello-api").remove(force=True)
```

### CLI

```bash
docker rm -f hello-api
```

### Python (Image optional entfernen)

```python
client.images.remove("school/hello-api:latest", force=True)
```

### CLI

```bash
docker rmi -f school/hello-api:latest
```

## 11) Bonus: Port wechseln (HOST_PORT=8081)

### Python

```python
ports={"8080/tcp": 8081}
```

### CLI

```bash
docker rm -f hello-api
docker run -d --name hello-api -p 8081:8080 -e WHO="World" -e PORT="8080" school/hello-api:latest
curl http://127.0.0.1:8081/
```

## Mini-Übersicht als „Spickzettel“

* Build: `docker build -t ... --build-arg ... .`
* History: `docker history ...`
* Labels: `docker inspect ... --format ...`
* Run Port+ENV: `docker run -d --name ... -p HOST:CONT -e ... IMAGE`
* Cleanup: `docker rm -f NAME` und `docker rmi -f IMAGE`
