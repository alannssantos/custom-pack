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
### Portainer: Gerencia os conteiners de forma grafica.
```bash
docker run -d \
  --name=portainer \
  -p 8000:8000 \
  -p 9000:9000 \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce
```
### Tranmission: Cliente torrent.
```bash
docker run -d \
  --name=transmission \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=America/Fortaleza \
  -p 9091:9091 \
  -p 51413:51413 \
  -p 51413:51413/udp \
  -v $HOME/Docker/transmission/config:/config \
  -v $HOME/Docker/transmission/downloads:/downloads \
  -v $HOME/Docker/transmission/watch:/watch \
  --restart unless-stopped \
  lscr.io/linuxserver/transmission:latest
```
### Sonarr: Pesquisar e baixar series com a ajuda de um cliente torrent.
```bash
docker run -d \
  --name=sonarr \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=America/Fortaleza \
  -p 8989:8989 \
  -v <path to tvshows>:/tv \
  -v <path to downloadclient-downloads>:/downloads \
  --restart unless-stopped \
  linuxserver/sonarr
```
### Radarr: Pesquisar e baixar filmes com a ajuda de um cliente torrent.
```bash
docker run -d \
 --name=radarr \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=America/Fortaleza \
  -p 7878:7878 \
  -v <path to movies>:/movies \
  -v <path to downloadclient-downloads>:/downloads \
  --restart unless-stopped \
  linuxserver/radarr
```
### Bazarr: Pesquisar por legendas para filmes e series do Radarr e Sonarr.
```bash
docker run -d \
  --name=bazarr \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=America/Fortaleza \
  -p 6767:6767 \
  -v $HOME/Docker/bazarr/config:/config \
  -v <path to tvshows>:/tv \
  -v <path to movies>:/movies \
  --restart unless-stopped \
  lscr.io/linuxserver/bazarr:latest
```
### JDownloader-2: Baixar arquivos de sites de hospedagens.
```bash
docker run -d \
  --name=jdownloader-2 \
  -p 5800:5800 \
  -v $HOME/Docker/jdownloader-2:/config:rw \
  -v $HOME/Downloads:/output:rw \
  --restart unless-stopped \
  jlesage/jdownloader-2
```
### Jellyfin: Servidor da biblioteca de filmes e series.
```bash
docker run -d \
  --name=jellyfin \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=America/Fortaleza \
  -p 8096:8096 \
  -v $HOME/Docker/jellyfin/library:/config \
  -v <path to tvshows>:/data/tvshows \
  -v <path to movies>:/data/movies \
  --restart unless-stopped \
  lscr.io/linuxserver/jellyfin:latest
```
### Ubooquity: Servidor de Livros e Mangas.
- Acesso a pagina de [Administrador](http://localhost:2203/ubooquity/admin)
- Acesso a pagina de [Usuario](http://localhost:2202/ubooquity)
```bash
docker run -d \
  --name=ubooquity \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=America/Fortaleza \
  -e MAXMEM=1024 \
  -p 2202:2202 \
  -p 2203:2203 \
  -v $HOME/Docker/ubooquity:/config \
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
