# Configuração para compartilhamento de jogos de PS2 pela rede.

```
[global]
  workgroup = WORKGROUP
  dns proxy = no
  smb ports = 445
  client min protocol = CORE
  client max protocol = NT1
  server min protocol = LANMAN1
  server max protocol = SMB3
  keepalive = 0
  lanman auth = yes
  null passwords = yes 

[PS2]
  comment = Jogos de PS2
  path = /media/Stronger/GAMES/PS2/
  browseable = yes
  read only = no
  guest ok = yes
  public = yes
  available = yes

```
