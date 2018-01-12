# cryptolilly
Cryptolilly - A personal documentation of a simple ethminer testing rig

## Ethereumpool Mining Stats
* Ethereumpool Public Key: 517e2a43fc6d1d4d0503a16db2490f8017080c9e
* [Ethereumpool Stats](https://ethereumpool.co/stats/miner/?address=0x517e2a43fc6d1d4d0503a16db2490f8017080c9e)


## Physical Location
In der Ati-Werkstatt Anorg H612. Schlüssel meist im ehemaligen Autoschlüsseldepot an der Türe.

## Network Location
Zugang via SSH auf Port 7222

```bash
ssh <username>@<anorg-h612-ip> -p 7222
```

## Mining

### Start Miner
Manuell starten, falls nötig.

```bash
cd /home/<username>/
./ethminer-boot.sh
```

### Autostart Miner
Poolmining auf ethereumpool.co startet automatisch nach Boot-Up (90 Sekunden Wartezeit). Dies damit sicher ist, dass Rechner am Netz hängt, ohne dass man dies via Script noch prüfen müsste.

```bash
cat /home/<username>/ethminer-boot.sh
```

```bash
#!/usr/bin/env bash
ps cax | grep ethminer > /dev/null
if [ $? -eq 0 ]; then
  echo "Ethminer is already running."
else
  echo "Ethminer being launched"
nohup /home/<username>/bin/ethminer  \
  -G -F http://ethereumpool.co/\?miner\=120@0x<POOL WALLET ADDRESS>@<OPTIONAL RIGNAME> \
        >> /var/log/miner.log 2>&1 </dev/null & echo $! > /home/<username>/tmp/miner.pid & sleep 10
fi
```


Simple Autostartintegration via crontab (nich via services!)
```bash
crontab -l
@reboot sleep 90 && bash /home/<username>/ethminer-boot.sh
```

### Stop/Kill Miner
Prozess-ID aus Autostart auslesen und terminieren.

```bash
cat /home/<username>/tmp/miner.pid
kill -9 <PID>
```

### Miner Monitoring & Logging
Für Monitoring & Debug. Log rotation ist aktiviert, d.h. ältere Logs werden nach einer Weile gelöscht.

```bash
#Ethminer Logging
tail -f /var/log/miner.log
```

### Hardware Monitoring (Temp./Fan)
lm_sensors ist installiert und minimal konfiguriert.
Temperatur der einzelnen Karten und Funktion Lüfter/Fan können so remote geprüft werden.

```bash
#/usr/bin/sensors
sensors

#OUTPUT
.
.
.
amdgpu-pci-0600
Adapter: PCI adapter
fan1:        1449 RPM
temp1:        +73.0°C  (crit =  +0.0°C, hyst =  +0.0°C)

amdgpu-pci-0400
Adapter: PCI adapter
fan1:        1234 RPM
temp1:        +73.0°C  (crit =  +0.0°C, hyst =  +0.0°C)

amdgpu-pci-0200
Adapter: PCI adapter
fan1:         730 RPM
temp1:        +73.0°C  (crit =  +0.0°C, hyst =  +0.0°C)
.
.
.
```

### Stromverbrauch
Kann derzeit am lokalen Messgerät gecheckt werden. Nach Deaktivierung aller unnötigen Features im BIOS verbraucht die Kiste im Mining-Betrieb mit 6 Karten 1050W.

## Updates & Installation
Wird ein neuer Kernel installiert, dann muss man die AMDGPU-PRO Treiber für Ubuntu 17.04 nochmals neu runterladen und installieren.

[Official Installation Instruction](http://support.amd.com/en-us/kb-articles/Pages/AMDGPU-PRO-Install.aspx)

```bash
dpkg -l amdgpu-pro
#Current Version: amdgpu-pro-17.40-492261

./amdgpu-pro-install --px
```

Unbeding mit --px installieren, da sonst GUI von Ubuntu nach Logging nicht funktioniert, da AMD und Intel On-Board Karten miteinander nicht klar kommen.


# Todos
* Prüfen ob Poolmining wirklich funktioniert, d.h. periodisch Ballance auf ethereumpool.co checken
* Mining Beta-Treiber von AMD mal ausprobieren (lokal und nicht übers Netz installieren, da Reboot via SSH ev. hängenbleibt). Performance-Gewinn ~5-10%
* GPU overclocking, da Karten eigentlich einiges mehr als 17-20M/Hs lierfern sollten.
* Andere Altcoins ausprobieren
