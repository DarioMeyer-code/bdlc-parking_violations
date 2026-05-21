# Cluster-Setup und Architektur

## Topologie

Für die Analyse der NYC Parking Violations wurde ein verteilter Hadoop-Cluster mit drei Knoten aufgesetzt. Der Cluster folgt einer Master-Slave-Architektur und besteht aus einem Master-Knoten und zwei Slave-Knoten.

| Rolle | Hostname |
|---|---|
| Master | `bdlc-012.bdlc.ls.eee.intern` |
| Slave 1 | `bdlc-005.bdlc.ls.eee.intern` |
| Slave 2 | `bdlc-018.bdlc.ls.eee.intern` |

Der Master-Knoten übernimmt sowohl koordinierende Aufgaben als auch Worker-Funktionen. Dies entspricht der Empfehlung aus den Kursunterlagen, alle verfügbaren Ressourcen für die Verarbeitung zu nutzen und nicht ungenutzt zu lassen.

## Service-Aufteilung

### Master-Knoten (bdlc-012)

Auf dem Master-Knoten laufen die folgenden Dienste:

- **NameNode** – verwaltet die Metadaten des verteilten Dateisystems HDFS und weiss, welcher Datenblock auf welchem DataNode liegt.
- **Secondary NameNode** – erstellt periodisch Checkpoints des NameNode-Edit-Logs als Backup-Mechanismus. Übernimmt kein automatisches Failover.
- **DataNode** – speichert HDFS-Blöcke. Der Master-Knoten wird damit gleichzeitig als Storage-Node genutzt.
- **Spark Master** – koordiniert die Spark Worker, verteilt Jobs und allokiert Executors.
- **Spark Worker** – führt zusätzlich Spark-Tasks aus, damit die Rechenressourcen des Master-Knotens nicht ungenutzt bleiben.
- **Spark History Server** – stellt abgeschlossene Spark-Jobs für Tuning und Debugging zur Verfügung.
- **JupyterLab** – Entwicklungsumgebung, in der die Notebooks für Pre-Processing, Prototyping und Analyse ausgeführt werden.

### Slave-Knoten (bdlc-005, bdlc-018)

Auf den beiden Slave-Knoten laufen jeweils:

- **DataNode** – Storage für HDFS-Blöcke. Mit einem Replikationsfaktor von zwei liegt jeder Datenblock doppelt im Cluster und ist somit gegen einen einzelnen Knotenausfall abgesichert.
- **Spark Worker** – führt die verteilten Spark-Tasks aus.

## Web-Oberflächen

Sämtliche Web-UIs sind auf dem Master-Knoten verfügbar und dienen zur Überwachung des Cluster-Zustands sowie zur Entwicklung:

| Service | URL | Zweck |
|---|---|---|
| HDFS NameNode | `http://bdlc-012.bdlc.ls.eee.intern:9870/` | Überblick über DataNodes, Browsen der Files, Replikationsstatus |
| Spark Master | `http://bdlc-012.bdlc.ls.eee.intern:8080/` | Worker-Übersicht, laufende Applications, Ressourcenauslastung |
| Spark History Server | `http://bdlc-012.bdlc.ls.eee.intern:18080/` | Abgeschlossene Jobs, Stages und Task-Statistiken |
| JupyterLab | `http://bdlc-012.bdlc.ls.eee.intern:8888/lab` | Entwicklungsumgebung für die Notebooks |

## Eingesetzte Frameworks und Tools

### HDFS (Hadoop Distributed File System)

HDFS dient als Speicher-Layer und legt sowohl die Rohdaten als auch die verarbeiteten Parquet-Daten ab. Das Dateisystem ist nach dem Master-Slave-Prinzip organisiert: Der NameNode hält die Metadaten, die DataNodes speichern die eigentlichen Blöcke mit einer Standardgrösse von 128 MB. Durch die Replikation auf zwei Knoten ist die Verfügbarkeit auch bei Ausfall eines Slaves gewährleistet. HDFS folgt dem Prinzip "Moving Computation is Cheaper than Moving Data" – Spark-Tasks werden möglichst auf jenen Knoten ausgeführt, auf denen die zugehörigen Datenblöcke liegen.

### Apache Spark

Apache Spark wird im Standalone-Modus für die gesamte Datenverarbeitung eingesetzt. Der Spark Master auf `bdlc-012` koordiniert die drei Spark Worker, die parallel auf den DataNode-Maschinen rechnen. Spark wurde gegenüber klassischem MapReduce gewählt, weil es Daten im Arbeitsspeicher verarbeitet und damit deutlich schneller iterative Analysen erlaubt. Die Interaktion erfolgt über PySpark in den JupyterLab-Notebooks.

### Apache Parquet

Die verarbeiteten Daten werden im Parquet-Format abgelegt. Parquet ist ein spaltenorientiertes, plattformübergreifendes Open-Source-Format, das mehrere Vorteile bietet: starke Komprimierung mittels Snappy reduziert das Datenvolumen typischerweise um den Faktor fünf bis zehn gegenüber CSV, das mitgespeicherte Schema entfällt beim Einlesen, und Spark kann gezielt einzelne Spalten von der Disk lesen, anstatt jeden Datensatz vollständig zu deserialisieren.

### Partitionierung

Zusätzlich werden die Parquet-Dateien nach `fiscal_year` partitioniert. Spark legt die Daten dabei in Unterordnern pro Jahr ab, wodurch Queries mit einem Filter auf das Fiskaljahr nur die relevanten Files lesen müssen. Dies ist eine bewährte Optimierungstechnik aus dem Hadoop-Ökosystem und beschleunigt zeitbezogene Analysen erheblich.

### JupyterLab

JupyterLab läuft als persistenter Service auf dem Master-Knoten und ist die zentrale Entwicklungsumgebung. Es erlaubt die Ausführung von PySpark-Code, das gleichzeitige Dokumentieren in Markdown-Zellen sowie die direkte Abgabe der Notebooks im geforderten `.ipynb`-Format.

## Begründung der Topologie-Wahl

Die Aufteilung mit einem Master und zwei Slaves wurde aus mehreren Gründen gewählt. Sie ist die einfachste Topologie, die ein echtes verteiltes Setup darstellt und damit den Bezug zu einem realen Big-Data-Cluster herstellt. Mit drei DataNodes und einem Replikationsfaktor von zwei ist gleichzeitig genug Redundanz vorhanden, um das Verhalten bei einem simulierten Knotenausfall realistisch zu testen. Der Master übernimmt zusätzlich Worker-Aufgaben, um die verfügbaren Rechenressourcen voll auszunutzen.

Das Framework-Set – HDFS für Storage, Spark für Compute, JupyterLab für die Entwicklung und Parquet als optimiertes Speicherformat – deckt den gesamten Big-Data-Workflow ab und entspricht der im Kurs vermittelten Standard-Architektur.
