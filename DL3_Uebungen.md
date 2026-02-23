# Übungsblatt DL 3 – Docker mit Python steuern

## Ziel

Du übst, wie du Docker **mit Python** bedienst:

* Verbindung prüfen
* Container starten und Ausgaben lesen
* Container im Hintergrund verwalten (starten/stoppen/löschen)

## Voraussetzungen

* Docker ist installiert und gestartet (Docker Desktop läuft oder Docker-Dienst läuft).
* Python 3 ist installiert.
* Python-Paket ist installiert:

```bash
pip install docker
```

# Aufgabe 1: Verbindung & Docker-Infos anzeigen ✅

## Auftrag

Erstelle eine Datei `aufgabe1_connection.py` und programmiere:

1. Python verbindet sich mit Docker.
2. Du gibst eine verständliche Meldung aus:

* ✅ wenn die Verbindung klappt
* ❌ wenn sie nicht klappt

3. Du gibst danach Infos aus: Docker-Version, Betriebssystem, Architektur.

## Start-Code (du darfst ihn übernehmen)

```python
import docker

def main():
    try:
        client = docker.from_env()
        if client.ping():
            print("✅ Verbindung zu Docker klappt.")
    except Exception as error:
        print("❌ Keine Verbindung zu Docker.")
        print("Tipp: Ist Docker gestartet?")
        print("Fehler:", error)
        return

    info = client.version()
    print("\n--- Docker-Infos ---")
    print("Docker-Version:", info.get("Version"))
    print("Betriebssystem:", info.get("Os"))
    print("Architektur:", info.get("Arch"))

if __name__ == "__main__":
    main()
```

## Ausführen

```bash
python aufgabe1_connection.py
```

## Notiere deine Ausgabe

* Docker-Version: _____________________________________________________________
* Betriebssystem: _____________________________________________________________
* Architektur: _________________________________________________________________

## Check-Fragen

1. Was bedeutet `True` bei `client.ping()`?
   ➡️ ______________________________________________________________________
2. Warum ist es sinnvoll, bei Fehlern eine Tipp-Meldung auszugeben?
   ➡️ ______________________________________________________________________

# Aufgabe 2: Einen Container starten und die Ausgabe verstehen

## Auftrag

Du startest einen Container, lässt ihn einen Befehl ausführen und liest die Ausgabe.

1. Lade das Image `ubuntu:latest` (falls es noch nicht da ist).
2. Starte einen Container, der Folgendes ausgibt:

* eine Begrüssung
* den aktuellen Benutzer im Container
* den Namen des Systems

## Beispiel-Code

Erstelle `aufgabe2_container_echo.py`:

```python
import docker

def main():
    client = docker.from_env()

    print("📥 Lade Image ubuntu:latest ...")
    client.images.pull("ubuntu:latest")
    print("✅ Image bereit.\n")

    command = "bash -lc 'echo Hallo aus dem Container!; whoami; uname -s'"
    output = client.containers.run(
        "ubuntu:latest",
        command,
        remove=True
    )

    print("--- Ausgabe aus dem Container ---")
    print(output.decode().strip())

if __name__ == "__main__":
    main()
```

## Ausführen

```bash
python aufgabe2_container_echo.py
```

## Notiere deine Ausgabe

Schreibe hier die 3 Zeilen ab, die du bekommst:

1. ---
2. ---
3. ---


## Check-Fragen

1. Erkläre in einem Satz: Was ist ein  Image?
   ➡️ _______________________________________________________
2. Erkläre in einem Satz: Was ist ein  Container?
   ➡️ _______________________________________________________
3. Was macht `remove=True`?
   ➡️ _______________________________________________________

# Aufgabe 3: Hintergrund-Container starten, finden, IP anzeigen, aufräumen

## Auftrag

Du startest einen Container im Hintergrund (der „läuft einfach kurz“), findest ihn wieder, liest Infos aus und räumst auf.

1. Starte `alpine:latest` im Hintergrund mit `sleep 30`.
2. Gib den Container-Namen aus.
3. Zeige alle laufenden Container an.
4. Zeige die IP-Adresse des Containers an.
5. Stoppe und lösche ihn.

## Beispiel-Code

Erstelle `aufgabe3_detach_ip.py`:

```python
import docker

def main():
    client = docker.from_env()

    print("📥 Lade Image alpine:latest ...")
    client.images.pull("alpine:latest")

    print("\n🚀 Starte Container im Hintergrund ...")
    container = client.containers.run(
        "alpine:latest",
        "sleep 30",
        detach=True
    )
    print("✅ Container-Name:", container.name)

    print("\n📋 Laufende Container:")
    for running_container in client.containers.list():
        print("-", running_container.name, "| Status:", running_container.status)

    container.reload()
    networks = container.attrs["NetworkSettings"]["Networks"]
    print("\n🌐 Netzwerk-Infos:")
    for network_name, network_data in networks.items():
        print("Netzwerk:", network_name)
        print("IP:", network_data.get("IPAddress"))

    print("\n🧹 Aufräumen ...")
    container.stop()
    container.remove()
    print("✅ Container gestoppt und gelöscht.")

if __name__ == "__main__":
    main()
```

## Ausführen

```bash
python aufgabe3_detach_ip.py
```

## Notiere deine Ergebnisse

* Container-Name: __________________________
* Netzwerk-Name: __________________________
* IP-Adresse: __________________________

## Check-Fragen

1. Warum ist `detach=True` praktisch?
   ➡️ _______________________________________________________
2. Was kann passieren, wenn du Container nie stoppst/löschst?
   ➡️ _______________________________________________________

## Bonus (freiwillig) ⭐

Ändere bei Aufgabe 2 das Image von `ubuntu:latest` auf eine feste Version, z. B.:

* `ubuntu:24.04` (falls verfügbar)

Frage: Siehst du Unterschiede in der Ausgabe oder beim Download?
➡️ _______________________________________________________
