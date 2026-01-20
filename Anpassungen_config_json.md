Empfohlene “Schalter” je Servertyp (nur 1 Zeile ändern)

Je nachdem, was du als Incident-Signal willst:

SQL-Server (meist korrekt):
Setze in Thresholds:

"DiskMode": "LatencyOrQueue"


Damit ignorierst du reinen Streaming-Durchsatz weitgehend und reagierst auf echte Storage-Pressure.

FILE-Server (Kopierjobs):
Eher:

"DiskMode": "AnyOf"


oder, wenn du sehr viele große Transfers hast:

"DiskMode": "ThroughputOnly"


RDS: AnyOf passt meistens, weil Disk-Latenz/Queue bei Profilen schnell auch User-Experience killt.

Wenn du willst, liefere ich dir zusätzlich vier separate Dateien (config.rds.json, config.file.json, config.sql.json, config.dc.json) mit je passenden DiskMode/Thresholds, damit du per Deployment nur die richtige Datei auswählst.