---
title: idempotence
tags: 1, 3days, 4days, 5days
---
# Idempotenz

### Definition Idempotenz
> Als idempotent bezeichnet man Vorgänge, Befehle oder Aktionen, die immer zum gleichen Ergebnis führen, egal wie oft sie ausgeführt werden.

Beispiel: Der Befehl `mkdir -p verzeichnis` hat als Ergebnis immer das angelegte Verzeichnis `verzeichnis`, egal wie oft er ausgeführt wurde.

### Beispiel für eine Anwendung des Prinzips Idempotenz in einem `bash`-Skript:
```bash=
.
.
# Variablen-Definition
tempverzeichnis=/tmp/"Temporäre Skriptdateien"/$(basename "$0")"
mkdir -p "$tempverzeichnis"
.
.<irgendein Befehl mit temporärem Speichern> > "$tempverzeichnis"
.
# Letzter Befehl
# rm -r "$tempverzeichnis"
```
### Beispiel kubernetes:
`kubectl create -f FILE.yaml` imperativ
`kubectl apply  -f FILE.yaml` deklarative - idempotent

### Gegen-Beispiele
- Dateiname mit Zeitstempel
- Datenbank-Migration

Ansible ist grundsätzlich idempotent.


