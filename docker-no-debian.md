# Debian 12
## Configuração básica para notebook-server.
### Baixar tela sem que ele desligue ou entre em suspensão.
`/etc/systemd/logind.conf`
```bash
HandlePowerKey=poweroff
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```
### Tela desliga depois de 5 minutos.
`/etc/default/grub`
```bash
GRUB_CMDLINE_LINUX="consoleblank=300"
```
### Atualizar configurações feitas.
`sudo systemctl restart systemd-logind.service`

`sudo update-grub`
### Ip Fixo. 
`/etc/network/interfaces`
```
# Configuração de IP Fixo.
auto enp2s0
allow-hotplug enp2s0
iface enp2s0 inet static
        address 192.168.0.132
        netmask 255.255.255.0
        network 192.168.0.1
        broadcast 192.168.0.255
        gateway 192.168.0.1
        dns-nameservers 1.1.1.1 8.8.8.8
```
## Instalação do Docker.
### Primeiro deve remover a antiga versão do Docker.
```bash
sudo apt remove \
  docker\
  docker.io \
  containerd \
  runc
```
### Atualize os repositórios.
```bash
sudo apt update
```
### Instale as dependências.
```bash
sudo apt install \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
```
### Adicione GPG Key Oficial.
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
### Adicione o repositório Docker.
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
### Instalar os binários.
```bash
sudo apt update
sudo apt install \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-compose-plugin
```
#### Se estiver dando erro no `sudo apt update`
```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
#### No caso de problema de permissão, provavelmente o usuário não esta no grupo docker, adicione-o e reinicie a maquina.
```bash
sudo usermod -a -G docker USUÁRIO
```
## Gerenciamento de contêineres.
### Watchtower: Agenda a atualização dos contêineres.
```bash
docker run -d \
  --name watchtower \
  -e TZ=America/Fortaleza \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower \
  --schedule "0 0 */6 * * *" \
  --debug --cleanup
```
### Homepage: Dashboard.
```bash
docker run -d \
  --name=homepage \
  -p 80:3000 \
  -v $HOME/.docker/homepage:/app/config \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v <path to monitor>:/mnt/storage \
  --restart unless-stopped \
  ghcr.io/gethomepage/homepage:latest
```
### Transmission: Cliente torrent.
```bash
docker run -d \
  --name=transmission \
  -e PUID=$(id -u) \
  -e PGID=$(id -g) \
  -e TZ=America/Fortaleza \
  -p 9091:9091 \
  -p 51413:51413 \
  -p 51413:51413/udp \
  -v $HOME/.docker/transmission:/config \
  -v $HOME/Downloads/Transmission:/downloads \
  --restart unless-stopped \
  lscr.io/linuxserver/transmission:latest
```
### Prowlarr: Indexador para Sonarr e Radarr
```bash
docker run -d \
  --name=prowlarr \
  -e PUID=$(id -u) \
  -e PGID=$(id -g) \
  -e TZ=America/Fortaleza \
  -p 9696:9696 \
  -v $HOME/.docker/prowlarr:/config \
  --restart unless-stopped \
  lscr.io/linuxserver/prowlarr:develop
```
### Sonarr: Pesquisar e baixar series com a ajuda de um cliente torrent.
```bash
docker run -d \
  --name=sonarr \
  -e PUID=$(id -u) \
  -e PGID=$(id -g) \
  -e TZ=America/Fortaleza \
  -p 8989:8989 \
  -v $HOME/.docker/sonarr:/config \
  -v <path to tvshows>:/tv \
  -v <path to download client-downloads>:/downloads \
  --restart unless-stopped \
  linuxserver/sonarr
```
### Radarr: Pesquisar e baixar filmes com a ajuda de um cliente torrent.
```bash
docker run -d \
  --name=radarr \
  -e PUID=$(id -u) \
  -e PGID=$(id -g) \
  -e TZ=America/Fortaleza \
  -p 7878:7878 \
  -v $HOME/.docker/radarr:/config \
  -v <path to movies>:/movies \
  -v <path to download client-downloads>:/downloads \
  --restart unless-stopped \
  linuxserver/radarr
```
### Bazarr: Pesquisar por legendas para filmes e series do Radarr e Sonarr.
```bash
docker run -d \
  --name=bazarr \
  -e PUID=$(id -u) \
  -e PGID=$(id -g) \
  -e TZ=America/Fortaleza \
  -p 6767:6767 \
  -v $HOME/.docker/bazarr:/config \
  -v <path to tvshows>:/tv \
  -v <path to movies>:/movies \
  --restart unless-stopped \
  lscr.io/linuxserver/bazarr:latest
```
### JDownloader 2: Baixar arquivos de sites de hospedagens.
```bash
docker run -d \
  --name=jdownloader-2 \
  -e PUID=$(id -u) \
  -e PGID=$(id -g) \
  -e TZ=America/Fortaleza \
  -p 5800:5800 \
  -v $HOME/.docker/jdownloader-2:/config:rw \
  -v $HOME/Downloads/JDownloader-2:/output:rw \
  --restart unless-stopped \
  jlesage/jdownloader-2
```
### Tachidesk: Pesquisa, baixa e servidor de mangas.
```bash
mkdir -p $HOME/.docker/tachidesk && \
docker run -d \
  --name=tachidesk \
  -e PUID=$(id -u) \
  -e PGID=$(id -g) \
  -e TZ=America/Fortaleza \
  -p 4567:4567 \
  -v $HOME/.docker/tachidesk:/home/suwayomi/.local/share/Tachidesk \
  --restart unless-stopped \
  ghcr.io/suwayomi/tachidesk:develop
```
### Jellyfin: Servidor da biblioteca de filmes e series.
```bash
docker run -d \
  --name=jellyfin \
  -e PUID=$(id -u) \
  -e PGID=$(id -g) \
  --net=host \
  -p 8096:8096 \
  -p 7359:7359/udp \
  -p 1900:1900/udp \
  -e TZ=America/Fortaleza \
  --device=/dev/dri:/dev/dri \
  -v $HOME/.docker/jellyfin:/config \
  -v <path to books>:/data/books \
  -v <path to mangas>:/data/mangas \
  -v <path to comics>:/data/comics \
  -v <path to movies>:/data/movies \
  -v <path to tvshows>:/data/tvshows \
  --restart unless-stopped \
  lscr.io/linuxserver/jellyfin:latest
```
### Jellyseerr: Catalogo de media (Opcional).
```bash
docker run -d \
  --name jellyseerr \
  -e LOG_LEVEL=debug \
  -e TZ=America/Fortaleza \
  -p 5055:5055 \
  -v $HOME/.docker/jellyseerr:/app/config \
  --restart unless-stopped \
  fallenbagel/jellyseerr:latest
```
### Ubooquity: Servidor de Livros e Mangas.
- Acesso a pagina de [Administrador](http://localhost:2203/ubooquity/admin)
- Acesso a pagina de [Usuário](http://localhost:2202/ubooquity)
```bash
docker run -d \
  --name=ubooquity \
  -e PUID=$(id -u) \
  -e PGID=$(id -g) \
  -e TZ=America/Fortaleza \
  -e MAXMEM=1024 \
  -p 2202:2202 \
  -p 2203:2203 \
  -v $HOME/.docker/ubooquity:/config \
  -v <path to books>:/books \
  -v <path to comics>:/comics \
  -v <path to raw files>:/files \
  --restart unless-stopped \
  lscr.io/linuxserver/ubooquity:latest
```
#### Saiba como remover um contêiner caso precise.
```bash
docker container rm CONTEINER
docker image rm REPOSITÓRIO
```
#### Saiba como atualizar todos os contêineres manualmente.
```bash
docker run --rm \
-v /var/run/docker.sock:/var/run/docker.sock \
containrrr/watchtower \
--run-once
```
