# Voraussetzung: IPv64 API-Key
Du brauchst den API-Key von IPv64 für die DNS-Challenge. Den findest du in deinem IPv64-Account unter Account → API-Key.
Installation & Cert beantragen
bash# acme.sh installieren
brew install acme.sh

# IPv64 API-Key setzen
export IPv64_Token="DEIN-API-KEY"

# Wildcard-Cert beantragen
acme.sh --issue \
  --dns dns_ipv64 \
  -d "2ms.home64.de" \
  -d "*.2ms.home64.de"
```

acme.sh setzt den TXT-Record automatisch bei IPv64, wartet auf Validierung und räumt danach auf.

## Was du bekommst
```
~/.acme.sh/2ms.home64.de/
├── 2ms.home64.de.cer       # Cert
├── 2ms.home64.de.key       # Private Key
├── ca.cer                  # Intermediate CA
└── fullchain.cer           # Cert + Chain (das brauchst du meistens)
Erneuerung
acme.sh richtet automatisch einen Cronjob ein – alle 60 Tage wird das Cert erneuert, solange der API-Key gesetzt ist.