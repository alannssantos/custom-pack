# Configuração para compartilhamento de jogos de PS2 pela rede.


Adicione este bloco em `[global]`.
```
#### Configuração Adicionais para PS2 ####
  client min protocol = CORE
  client max protocol = NT1
  server min protocol = LANMAN1
  server max protocol = SMB3
```
```
[PS2]
  comment = Jogos de PS2
  path = /media/Stronger/GAMES/PS2/
  browseable = yes
  read only = no
  guest ok = yes
  public = yes
  available = yes
```
