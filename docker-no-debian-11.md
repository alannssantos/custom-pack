# Debian 11
## Instalação do Docker
### Primeiro deve remover a antiga versão do Docker.
```bash
sudo apt remove \
  docker\
  docker-engine \
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
## Gerenciamento de conteiners.
### Watchtower: Agenda a atualizão dos conteiners.
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
  ghcr.io/benphelps/homepage:latest
```
### Tranmission: Cliente torrent.
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
### pyLoad: Baixar arquivos de sites de hospedagens.
```bash
docker run -d \
  --name=pyload-ng \
  -e PUID=$(id -u) \
  -e PGID=$(id -g) \
  -e TZ=America/Fortaleza \
  -p 8000:8000 \
  -v $HOME/.docker/pyload:/config \
  -v $HOME/Downloads/pyLoad:/downloads \
  --restart unless-stopped \
  lscr.io/linuxserver/pyload-ng:latest
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
### Ubooquity: Servidor de Livros e Mangas.
- Acesso a pagina de [Administrador](http://localhost:2203/ubooquity/admin)
- Acesso a pagina de [Usuario](http://localhost:2202/ubooquity)
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
#### Saiba como remover um conteiner caso precise.
```bash
docker container rm CONTEINER
docker image rm REPOSITÓRIO
```
