# WinPerfWatch

## Starten als Consolenanwendung  
winperfwatch.exe -c  

## Installieren als WIndows Service
winperfwatch.exe -s

## Deinstallieren des Windows Service
winperfwatch.exe -u

## Logfiles
./logs/

## Konfigurationsdatei
./config.json

## NLog Konfigruationsdatei
./NLog.config

## Chatgpt Link
https://chatgpt.com/c/696136c0-1e68-8326-8252-d3584e479ffa

## Werte als Exponentially Weighted Moving Average (EWMA)
CPU: Prefer “raw” + geglättete Werte  
Glättung per EWMA (Exponentially Weighted Moving Average) pro Metrik, z. B. alpha=0.3.  
Incidents auf geglätteten Werten entscheiden, Rohwerte weiterhin loggen.  
Umsetzung: Speichere pro Metrik ewmaCpu, ewmaDisk, ewmaMem. Logge beides:  
cpuPctRaw, cpuPctEwma etc  
  
Der Exponentially Weighted Moving Average (EWMA) ist ein gleitender Durchschnitt, bei dem neuere Werte stärker gewichtet werden als ältere.  
Das macht ihn reaktionsschneller als einen einfachen gleitenden Durchschnitt (SMA).  

Beispiel 1: Schritt für Schritt  
  
Messwerte (z. B. CPU-Auslastung in %):  
  
Zeit:   1   2   3   4  
Wert:  10  20  30  40  
  
    
Wir wählen α = 0,3  
Startwert: EWMA₁ = 10  

EWMA₂ = 0,3 · 20 + 0,7 · 10 = 13  
  
EWMA₃ = 0,3 · 30 + 0,7 · 13 = 18,1  
  
EWMA₄ = 0,3 · 40 + 0,7 · 18,1 = 24,67  
  
Beobachtung:  
Der EWMA folgt dem Trend, springt aber nicht abrupt.  

