## Server Specs
Als Server habe ich ein Server mit 2 Intel CPU Cores und 4 GB RAM ausgewählt. Als Image habe ich ein custom Image ausgewählt, welches ich einmal vor langer zeit erstellt habe. Es verwendet als OS Debian 11 und beinhaltet die wichtigsten Konfigurationen, welche man grudsätzlich auf allen Server haben sollte wie z.B. Konfigurierte Non Root User mit `SSH` Key usw. Durch dieses custom Image kann ich relativ schnell Server erstellen ohne grösseren Aufwand

Der Server ferfügt sowohl über eine Öffentliche IP wie allerdings auch über eine Private IP mit der er später mit den anderen Nodes komunizieren kann.
## Updaten des Servers

Was ich allerdings troz des custom Images noch machen muss ist den Server einmal zu updaten. Sol läuft er z.B. immernoch auf den Linuxkernal Version `5.10.0-19-amd64` der neuste Kernal der zum aktuellen Zeitpunkt für Debian 11 zur Verfügung steht ist `5.10.0-21-amd64`.
```
$uname -a
Linux RancherNode1 5.10.0-19-amd64 #1 SMP Debian 5.10.149-2 (2022-10-21) x86_64 GNU/Linux
```bash
Command um den Server upzudaten und neuzustarten
```bash
$sudo apt update && sudo apt full-upgrade -y && sudo reboot
```
### Installation
Als erstes brauche ich natürlich eine Docker Engine. Dafür verwende ich das `docker.io` Package welches von den offizielen Debian Repositorys zur Verfügung gestellt wird

```bash
$sudo apt install docker.io -y
```
