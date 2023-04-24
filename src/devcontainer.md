# How to get started

## Installation von Docker Desktop
Als allererstes müssen Sie sich Docker Desktop für ihr Betriebssystem herunterladen. Am besten gehen Sie dazu auf die [offizielle Website von Docker](https://www.docker.com/products/docker-desktop/). Anschliessend klicken Sie die Installation durch und warten anschliessend, bis diese fertig ist. <br><br>

## Installation von Git
Als Nächstes müssen Sie sich Git installieren, um danach das Github Repository zu clonen und um später auf Github commiten & pushen zu können. Dazu können Sie auf die [offizielle Website von Git](https://git-scm.com/downloads) gehen und sich die Version für Ihren Computer herunterladen. Hier klicken Sie erneut die Installation durch und warten erneut, bis diese fertig ist. <br><br>

## Github Repo mit DevContainer clonen
Im nächsten Schritt können Sie sich das Github Repository von Github holen. Das machen Sie, indem Sie als Erstes im Explorer in den Ordner wechseln, in dem Sie das Repository haben möchten. Anschliessen können sie in diesem Ordner die Commandline öffnen (`Windows -> CMD` / `Linux -> Terminal` / `MacOS -> Terminal`) und dort den folgenden Befehl eingeben:

```bash
git clone https://github.com/Yanni8/M347-DevContainer.git
```
<br>

# Erste Schritte mit Docker

## Docker Container Starten
Um den Devcontainer starten zu können, müssen Sie sich zuerst noch eine Extension in VSC herunterladen. Dazu gehen Sie in VSC in die Extensions, suchen dort "Dev Container" und laden sich diese Extension herunter. <br>
Man darf umbedingt auch nicht vergessen das `.env` file mit allen nötigen Variablen zu füllen. Diese sind auffindbar im `.env.example.local` 
Sobald Sie das erledigt haben, können Sie in VSC auf `File` -> `Open Folder` -> Ordner mit dem Devcontainer auswählen und anschliessend das Main-File öffnen. Sobald Sie das gemacht haben, können Sie in VSC unten links auf das grüne Rechteck klicken und dann auf `Reopen in container` <br>
Jetzt können Sie in VSC unter `Terminal` ein neues Terminal-Fenster öffnen und mit dem Docker Container arbeiten. <br>

![image](https://user-images.githubusercontent.com/103995309/233073457-61db90c5-a63a-4cc0-89c7-ea607350ddeb.png) <br>

© 2023 Yannick Mueller, Ewan Joseph Waldmann, Levin Schaller
