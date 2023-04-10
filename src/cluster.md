# Setup Rancher on Server

## Server Specs
Als Server habe ich ein Server mit 2 Intel CPU Cores und 4 GB RAM ausgewählt. Als Image habe ich ein custom Image ausgewählt, welches ich einmal vor langer zeit erstellt habe. Es verwendet als OS Debian 11 und beinhaltet die wichtigsten Konfigurationen, welche man grudsätzlich auf allen Server haben sollte wie z.B. Konfigurierte Non Root User mit `SSH` Key usw. Durch dieses custom Image kann ich relativ schnell Server erstellen ohne grösseren Aufwand

Der Server ferfügt sowohl über eine Öffentliche IP wie allerdings auch über eine Private IP mit der er später mit den anderen Nodes komunizieren kann.
## Updaten des Servers

Was ich allerdings troz des custom Images noch machen muss ist den Server einmal zu updaten. Sol läuft er z.B. immernoch auf den Linuxkernal Version `5.10.0-19-amd64` der neuste Kernal der zum aktuellen Zeitpunkt für Debian 11 zur Verfügung steht ist `5.10.0-21-amd64`.
```bash
$uname -a
Linux RancherNode1 5.10.0-19-amd64 #1 SMP Debian 5.10.149-2 (2022-10-21) x86_64 GNU/Linux
```
Command um den Server upzudaten und neuzustarten
```bash
$sudo apt update && sudo apt full-upgrade -y && sudo reboot
```
### Installation
Als erstes brauche ich natürlich eine Docker Engine. Dafür verwende ich das `docker.io` Package welches von den offizielen Debian Repositorys zur Verfügung gestellt wird

```bash
$sudo apt install docker.io apparmor -y
```

Als nächstes habe ich einen DNS request gemacht, welcher mit der Domain `rancher.bbzbl-it.dev` auf die öffentliche IP vom gerade eben genannten Server zeigt. 


#### Starten des Management Clusters
Als erste musste ich das Management Cluster starten. Dieses ist dafür zuständig die verschiedensten Cluster zu managen, zu überwachen und stellt auch die Rancher UI zur Verfügung. Theoretisch kann man auch mehrere Server zu einem Managementcluster verbinden, um die Ausfahlsicherheit zu erhöhen. Da allerdings der Ausfall vom Managementcluster keinen Direckten impact auf die deployten applicationen hat habe ich mich aus Kostengründen dazu etnschieden das Managementcluster auf eier einzelnen Node zu deployen.

Um das Managementcluster einfach zu deployen verwende ich das [Offizielle Docker Image](https://hub.docker.com/r/rancher/rancher) von Rancher. Dieses deployt alle nötigen Container und das singel Node Kubernetscluster in einem einzelnen Dockercontainer. Da das deployen eines Kubernetscluster direckten Zugriff auf den Linux Kernal benötigt muss ich den Dockercontainer mit der `--privileged` Flag starten. Dadurch fällt aber auch eine gewisse Isolation zwischen dem Dockercontainer und dem Hostsystem weg. Spricht wenn ein Angreiffer Zugriff auf den DOcker container hat kann er auch einfach das ganze System komprimieren.

```
docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    --privileged \
    rancher/rancher:latest \
    --acme-domain rancher.bbzbl-it.dev
```

#### Erstes einloggen

Nach dem der Dockercontainer nach etwa 15 Sekunden vollständig aufgestartet war konnte ich das random generierte initiale Password ganz einfach durch die Logs auslesen.
```bash
sudo docker logs  $(sudo docker ps -q)  2>&1 | grep "Bootstrap Password:"
```
<br/>
Nach dem ersten einloggen auf der Webui wird man dann anschliessend dazu aufgefordert, dass Password abzuändern. Das habe ich dann natürlich auch gemacht. Ebenfalls ist es **wichtig** dass man den Hacken unkreuzt, dass man sich dazu bereit erklärt Anonyme Nutzerdaten zu senden. Natürlich ist es jedem selber überlassen ob man Anonyme Daten senden möchte. Allerdings muss man es explizit **nicht** ankreuzen da es per default angekreuzt ist.

![Screenshot from 2023-04-09 14-34-57](https://user-images.githubusercontent.com/99135388/230933329-e8228e9e-38c4-4feb-97ac-dced13339abe.png)


#### Erstellen eines ersten Clusters

Nachdem ich mich eingeloggt habe, habe ich als erstes ein neues Cluster erstellt. Auf diesem Cluster beabsichtige ich dann alle Resourcen zu Deployen. Um einfach Clusters zu erstellen kann man sogennante [Node Drivers](https://ranchermanager.docs.rancher.com/v2.5/how-to-guides/advanced-user-guides/authentication-permissions-and-global-configuration/about-provisioning-drivers/manage-node-drivers) verweden. Diese können je nach Anbieter konfiguriert werden und erstellen automatisiert Server und überwachen diese auch. 
<br/>

![Screenshot from 2023-04-09 14-36-35](https://user-images.githubusercontent.com/99135388/230936188-2e46472e-4939-4a4e-ace8-4fdff9701e26.png)

<br/>
Leider ist per default kein Node Driver für meinen Cloud Anbieter vorhanden. Aus diesem Grund musste ich zuerst im Internet nach einem passenden Suchen. Und bin schliesslich auch [fundig geworden](https://github.com/mxschmitt/ui-driver-hetzner)
Das Hinzufügen des Drivers verlief dann relativ schnell und unkomliziert. Alles was ich machen musste war, dass ich die URL angeben musste, mit welcher der Node Driver hinuntergeladen werden konnte.

![Screenshot from 2023-04-09 14-38-01](https://user-images.githubusercontent.com/99135388/230936326-86a02f62-1493-45f2-b9df-698accda67a2.png)

<br/>

Nach dem Hnzufügen konnte ich einfach auf der View `Cluster Erstellen` meinen Cloud Provider auswählen. Ich fand es auch cool, dass es Support für VSphere gab. Dadurch könnte man auch ganz einfach in einer Private Cloud (Wenn man VSphere verwendet) mithilfe von Node Drivers ein eigenes Kubernetscluster aufbauen. 

![Screenshot from 2023-04-09 14-38-39](https://user-images.githubusercontent.com/99135388/230937141-45e7b4ec-11fa-4647-b12c-890ac1369546.png)

Nachher musste ich zuerst bei meinem Cloud provider ein `Privates Network` und eine `Placement Group` erstellen. Zwar ist dieser Schritt optional allerdings bietet er grosse Vorteile. Ein `Privates Network` sorgt dafür, dass bestimmte möglicherweise sensitive Ports nicht öffentliche erreichbar sind und zudem traffic nicht mitgeschnitten werden kann (auch wenn theoretisch jeglicher Traffic encryptet sein sollte). Eine `Placement Group` sorgt dafür, dass die Server Physikalisch und Logisch voneiander unabhängig sind. So würde z.B. der Ausfall von enem Networkswitch nur dazu führen dass 1 und icht gleich alle 3 Nodes nichtmehr erreichbar sind. 


![Screenshot from 2023-04-09 14-43-49](https://user-images.githubusercontent.com/99135388/230937059-e84c8bc0-50fc-4613-afc8-a48dd9b8da96.png)
