# Setup Rancher on Server

## Server Specs
Als Server wurde ein Server mit 2 Intel CPU Cores und 4 GB RAM ausgewählt. Als Image wurde ein custom Image ausgewählt, welches bereits vor längerer Zeit selber erstellt wurde. Es verwendet Debian 11 als OS und beinhaltet die wichtigsten Konfigurationen, welche man grundsätzlich auf allen Server haben sollte, wie z.B. konfigurierte Non Root User mit `SSH Key` usw. Durch dieses custom Image kann man relativ schnell Server erstellen und das ohne grösseren Aufwand.

Der Server verfügt sowohl über eine öffentliche IP, als auch über eine private IP, mit der er später mit den anderen Nodes kommunizieren kann. <br><br>

## Updaten des Servers
Was allerdings trotz des custom Images noch gemacht werden muss, ist den Server einmal zu updaten. So läuft er z.B. immer noch auf der Linuxkernal Version `5.10.0-19-amd64`, der neuste Kernal der zum aktuellen Zeitpunkt für Debian 11 zur Verfügung steht ist `5.10.0-21-amd64`.

```bash
$uname -a
Linux RancherNode1 5.10.0-19-amd64 #1 SMP Debian 5.10.149-2 (2022-10-21) x86_64 GNU/Linux
```
Command, um den Server upzudaten und neuzustarten.

```bash
$sudo apt update && sudo apt full-upgrade -y && sudo reboot
```
<br>

## Installation
Als Erstes wird natürlich eine Docker Engine benötigt. Dafür wurde das `docker.io` Package verwendet, welches von den offiziellen Debian Repositorys zur Verfügung gestellt wird.

```bash
$sudo apt install docker.io apparmor -y
```

Als Nächstes wurde einen DNS Request gemacht, welcher mit der Domain `rancher.bbzbl-it.dev` auf die öffentliche IP vom gerade eben genannten Server zeigt. <br><br>

## Starten des Management-Clusters
Als erste musste das Management-Cluster gestartet werden. Dieses ist dafür zuständig, die verschiedensten Cluster zu managen, zu überwachen und stellt auch die Rancher UI zur Verfügung. Theoretisch kann man auch mehrere Server zu einem Management-Cluster verbinden, um die Ausfallsicherheit zu erhöhen. Da allerdings der Ausfall vom Management-Cluster keinen direkten Einfluss auf die deployten Applikationen hat, wurde aus Kostengründen entschieden, das Management-Cluster auf einer einzelnen Node zu deployen.

Um das Management-Cluster einfach zu deployen, wurde das [offizielle Docker Image](https://hub.docker.com/r/rancher/rancher) von Rancher verwendet. Dieses deployt alle nötigen Container und das single Node Kubernetescluster in einem einzelnen Dockercontainer. Da das Deployen eines Kubernetescluster direkten Zugriff auf den Linux Kernal benötigt, muss man den Dockercontainer mit der `--privileged` Flag starten. Dadurch fällt aber auch eine gewisse Isolation zwischen dem Dockercontainer und dem Hostsystem weg. Heisst, wenn ein Angreifer Zugriff auf den Dockercontainer hätte, könnte er auch einfach das ganze System komprimieren.

```bash
$sudo docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    --privileged \
    rancher/rancher:latest \
    --acme-domain rancher.bbzbl-it.dev
```
<br>

## Erstes Einloggen

Nachdem der Dockercontainer nach etwa 15 Sekunden vollständig aufgestartet war, konnte das random generierte initiale Password ganz einfach durch die Logs ausgelesen werden.

```bash
$sudo docker logs  $(sudo docker ps -q)  2>&1 | grep "Bootstrap Password:"
```

Nach dem ersten Einloggen auf der Web-UI, wird man anschliessend dazu aufgefordert, das Passwort abzuändern. Ebenfalls ist es **wichtig**, dass man den Haken wegnimmt, bei dem steht, dass man sich dazu bereit erklärt, anonyme Nutzerdaten zu senden. Natürlich ist es jedem selber überlassen, ob man anonyme Daten senden möchte. Allerdings muss man es explizit **nicht** ankreuzen, da es per Default angekreuzt ist. <br><br>

![Screenshot from 2023-04-09 14-34-57](https://user-images.githubusercontent.com/99135388/230933329-e8228e9e-38c4-4feb-97ac-dced13339abe.png) <br><br>

## Erstellen eines ersten Clusters

Nach dem Einloggen, wurde als erstes ein neues Cluster erstellt. Auf diesem Cluster wird beabsichtigt, alle Ressourcen zu deployen. Um einfach Clusters zu erstellen, kann man sogenannte [Node Drivers](https://ranchermanager.docs.rancher.com/v2.5/how-to-guides/advanced-user-guides/authentication-permissions-and-global-configuration/about-provisioning-drivers/manage-node-drivers) verwenden. Diese können je nach Anbieter konfiguriert werden und erstellen automatisiert Server und überwachen diese auch. <br><br>

![Screenshot from 2023-04-09 14-36-35](https://user-images.githubusercontent.com/99135388/230936188-2e46472e-4939-4a4e-ace8-4fdff9701e26.png) <br><br>

Leider ist per Default kein Node Driver für unseren Cloud-Anbieter vorhanden. Aus diesem Grund wurde zuerst im Internet nach einem passenden gesucht. Schlussendlich wurde auch ein [Node Driver](https://github.com/mxschmitt/ui-driver-hetzner) gefunden.
Das Hinzufügen des Drivers verlief dann relativ schnell und unkompliziert. Alles was gemacht werden musste war, dass man die URL angeben musste, mit welcher der Node Driver heruntergeladen werden konnte.<br><br>

![Screenshot from 2023-04-09 14-38-01](https://user-images.githubusercontent.com/99135388/230936326-86a02f62-1493-45f2-b9df-698accda67a2.png) <br><br>

Nach dem Hinzufügen konnte einfach auf der View `Cluster Erstellen` unser Cloud-Provider ausgewählt werden. Es war auch cool, dass es Support für VSphere gab. Dadurch könnte man auch ganz einfach in einer private Cloud (wenn man VSphere verwendet), mithilfe von Node Drivers ein eigenes Kubernetescluster aufbauen. <br><br>

![Screenshot from 2023-04-09 14-38-39](https://user-images.githubusercontent.com/99135388/230937141-45e7b4ec-11fa-4647-b12c-890ac1369546.png) <br><br>

Später musste bei unserem Cloud-Provider ein `Privates Network` und eine `Placement Group` erstellt werden. Zwar ist dieser Schritt optional, allerdings bietet er grosse Vorteile. Ein `Privates Network` sorgt dafür, dass bestimmte, möglicherweise sensitive Ports, nicht öffentlich erreichbar sind und zudem Traffic nicht mitgeschnitten werden kann (auch wenn theoretisch jeglicher Traffic encryptet sein sollte). Eine `Placement Group` sorgt dafür, dass die Server physikalisch und logisch voneinander unabhängig sind. So würde z.B. der Ausfall von einem Networkswitch nur dazu führen, dass ein und nicht gleich alle drei Nodes nicht mehr erreichbar sind. <br><br>
Nachdem dieser Schritt erledigt war, wurde über die UI das Kluster initialisiert. <br><br>

![Screenshot from 2023-04-09 14-43-49](https://user-images.githubusercontent.com/99135388/230937059-e84c8bc0-50fc-4613-afc8-a48dd9b8da96.png) <br><br>

Anschliessend dauerte es etwa 10 Minuten, bis alle Server korrekt eingerichtet waren und das Cluster erreichbar war. <br><br>

![Screenshot from 2023-04-09 16-05-14](https://user-images.githubusercontent.com/99135388/230939681-55190075-38a7-44a8-bc71-ea9f95b813e8.png) <br><br>

![Screenshot from 2023-04-09 16-04-11](https://user-images.githubusercontent.com/99135388/230948634-5932c7ea-21b3-4460-95be-8a3a03b812fa.png) <br><br>

Danach wurde ein Test `Nginx` Container deployt. Das Deployen des Containers und das Erstellen eines Ingres verlief ohne grössere Probleme. Leider stellte sich etwas später heraus, dass der Ingres jedes Mal, nachdem man ihn anspricht, timeouted. Laut dem Internet liegt das häufig daran, dass der Port `8472/UDP` nicht geöffnet ist. Also wurde `nmap` verwendet, um herauszufinden, ob der Port wirklich nicht offen ist.

```bash
$nmap -p8472 -sU <IP>
Starting Nmap 7.80 ( https://nmap.org )
Nmap scan report for ****
Host is up (0.0010s latency).

PORT     STATE         SERVICE
8472/udp closed otv

Nmap done: 1 IP address (1 host up) scanned in 0.36 seconds
``` 
<br>

Und tatsächlich stellte sich heraus, dass der Port geschlossen war. Nach einer etwas längeren Recherche stellte sich heraus, dass für die Kommunikation das Netzwerkinterface `eth0` verwendet wird, sofern nichts angegeben wurde. Aus diesem Grund musste in der Netzwerkkonfiguration explizit angeben werden, welches Netzwerk verwendet werden sollte. Nach einem Restart der verschiedenen Nodes hat dann auch der Ingres funktioniert.

```yml
  network:
    canal_network_provider:
      iface: <Interface>
    flannel_network_provider:
      iface: <Interface>
```
<br>

Nachdem nun das Cluster voll erreichbar war, konnte nun auch damit anfangen werden, unsere ersten Projekte zu deployen. Wie bereits gesagt, sollte das Cluster dafür verwendet werden. Um das zu realisieren, wurden einfach die Projekte dockerisiert und anschliessend wurde das Image auf ein Repository gepusht, um es anschliessend auf das Cluster zu pullen. <br>
Etwas, was wir allerdings noch nie gemacht haben ist, die ganzen Images als Non Root User auszuführen. In den meisten Fällen wird die Applikation bereits von einem Non Root User ausgeführt. Die einzige Ausnahme dafür stellen unsere `nginx` Container dar. Glücklicherweise musste lediglich das Baseimage von `nginx` auf [`nginx-unpriviliged`](https://hub.docker.com/r/nginxinc/nginx-unprivileged) abgeändert werden, um sie als Non Root User zu starten. <br><br>

## Absichern der Nodes
Da unser Cluster nun funktionierte, wurden die Nodes gesichert. Eine Firewall hatten wir ja bereits. Das heisst, dass wir uns gleich daran machen konnten, einen `non-root` User zu erstellen, das Login für den `root` User zu deaktivieren usw.
