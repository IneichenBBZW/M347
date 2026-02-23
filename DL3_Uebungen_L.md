# Lösungen – Übungsblatt

Docker mit Python steuern

## ✅ Lösung zu Aufgabe 1

**Verbindung prüfen und Docker-Infos anzeigen**

### Musterlösung `aufgabe1_connection.py`

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

### Beispiel-Ausgabe

```
✅ Verbindung zu Docker klappt.

--- Docker-Infos ---
Docker-Version: 29.2.1
Betriebssystem: linux
Architektur: amd64
```

### Musterantworten Check-Fragen

1) Was bedeutet `True` bei `client.ping()`?
   ➡️ Python kann den Docker-Server erreichen und Docker läuft.
2) Warum ist eine verständliche Fehlermeldung sinnvoll?
   ➡️ Damit Benutzer sofort wissen, was sie prüfen müssen (z. B. Docker starten).

## ✅ Lösung zu Aufgabe 2

**Container starten und Ausgabe verstehen**

### Musterlösung `aufgabe2_container_echo.py`

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

### Beispiel-Ausgabe

```
Hallo aus dem Container!
root
Linux
```

### Musterantworten Check-Fragen

1) Was ist ein Image?
   ➡️ Ein Image ist eine Vorlage, aus der Container erstellt werden.
2) Was ist ein Container?
   ➡️ Ein Container ist ein gestartetes Programm, das aus einem Image entsteht.
3) Was macht `remove=True`?
   ➡️ Der Container wird nach dem Ausführen automatisch gelöscht.

## ✅ Lösung zu Aufgabe 3

Hintergrund-Container, IP anzeigen, aufräumen

### Musterlösung `aufgabe3_detach_ip.py`

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

### Beispiel-Ausgabe

```
📥 Lade Image alpine:latest ...

🚀 Starte Container im Hintergrund ...
✅ Container-Name: friendly_pike

📋 Laufende Container:
- friendly_pike | Status: running

🌐 Netzwerk-Infos:
Netzwerk: bridge
IP: 172.17.0.2

🧹 Aufräumen ...
✅ Container gestoppt und gelöscht.
```

### Musterantworten Check-Fragen

1) Warum ist `detach=True` praktisch?
   ➡️ Der Container läuft im Hintergrund weiter und blockiert das Programm nicht.

2) Was passiert, wenn Container nie gelöscht werden?
   ➡️ Sie verbrauchen Speicher und Ressourcen und machen das System unübersichtlich.

## ⭐ Bonus – Musterlösung

### Änderung

```python
client.images.pull("ubuntu:24.04")
```

### Erwartete Beobachtung

* Grösserer oder anderer Download
* Gleiche Befehle funktionieren weiterhin

### Musterantwort

➡️ Unterschiedliche Versionen können unterschiedliche Programme oder Versionen enthalten.
