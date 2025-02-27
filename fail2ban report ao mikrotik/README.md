# Primeiramento se você tem o fail2ban, certamente tem o python.
# Então iremos precisar de alguns modulos do python 
```
pip3 install paramiko ipaddress routeros_api
```

# baixe o arquivo blackhole-ban na pasta /usr/bin
```
wget https://raw.githubusercontent.com/joandson19/Fail2Ban/refs/heads/main/fail2ban%20report%20ao%20mikrotik/blackhole-ban -O /usr/bin/blackhole-ban
chmod +x /usr/bin/blackhole-ban
```
# Crie o arquivo mkblackhole.conf na pasta /etc/fail2ban/action.d/ e coloque o script abaixo.
```
[Definition]
actionban   = blackhole-ban add <ip>
actionunban = blackhole-ban remove <ip>
actioncheck =
actionstart =
actionstop =

[Init]
blocktype = blackhole
```

# Altere as linhas de banaction e banaction_allportes no arquivo jail.conf na pasta /etc/fail2ban
```
banaction = mkblackhole
banaction_allports = mkblackhole
```

# Crie um usuario e senha no mikrotik
```
/user add address="RANGE IP AUTORIZADA" group=full name="USUARIO FAIL2BAN" password="SENHA"

```

# Reinicie o fail2ban
```
service fail2ban restart
```

## Se seguiu tudo a risca, então é só conferir os logs do seu mikrotik como no exemplo abaixo.
![Captura de tela 2023-09-01 095639](https://github.com/joandson19/Fail2Ban/assets/36518985/863e14b1-ab9a-428a-87e5-0f2f5dbf6bcf)
