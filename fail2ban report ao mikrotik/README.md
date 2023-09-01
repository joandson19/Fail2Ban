# Primeiramento se você tem o fail2ban, certamente tem o python.
# Então iremos precisar de alguns modulos do python 
```
pip install paramiko ipaddress
```

# Crie o arquivo blackhole-ban na pasta /usr/bin
```
nano /usr/binblackhole-ban
```
* Dentro do arquivo cole o script abaixo
```
#!/bin/python3

import paramiko
import sys
import ipaddress

# Defina as informações de conexão SSH com o dispositivo MikroTik
router_ip = 'IP DO MIKROTIK'
router_username = 'USUARIO FAIL2BAN'
router_password = 'SENHA'

# Defina as informações da sua rede local com múltiplos prefixos
redes_locais = [
    ipaddress.IPv4Network('192.168.1.0/24'),
    ipaddress.IPv4Network('10.0.0.0/8'),
    ipaddress.IPv4Network('172.16.0.0/12')
]
def add_blackhole(ip_address):
    try:
        # Conecte-se ao dispositivo MikroTik via SSH
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh.connect(router_ip, username=router_username, password=router_password)

        # Verifique se o IP fornecido pertence a uma rede local
        if is_local_ip(ip_address):
            print(f"O IP {ip_address} pertence a uma rede local e não pode ser adicionado à lista de blackhole.")
        else:
            # Execute o comando para adicionar o IP à lista de blackhole
            command = f"/ip route add dst-address={ip_address} blackhole"
            ssh.exec_command(command)
            print(f"O IP {ip_address} foi adicionado à lista de blackhole com sucesso.")
    except Exception as e:
        print(f"Erro ao adicionar o IP {ip_address} à lista de blackhole: {str(e)}")
    finally:
        ssh.close()

def remove_blackhole(ip_address):
    try:
        # Conecte-se ao dispositivo MikroTik via SSH
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh.connect(router_ip, username=router_username, password=router_password)

        # Execute o comando para remover o IP da lista de blackhole
        command = f"/ip route remove [find dst-address={ip_address}]"
        ssh.exec_command(command)
        print(f"O IP {ip_address} foi removido da lista de blackhole com sucesso.")
    except Exception as e:
        print(f"Erro ao remover o IP {ip_address} da lista de blackhole: {str(e)}")
    finally:
        ssh.close()

def is_local_ip(ip_address):
    try:
        ip = ipaddress.IPv4Address(ip_address)
        for rede_local in redes_locais:
            if ip in rede_local:
                return True
        return False
    except ValueError:
        return False

if __name__ == '__main__':
    if len(sys.argv) != 3:
        print("Uso: python script.py add/remove IP_ADDRESS")
        sys.exit(1)

    action = sys.argv[1]
    ip_address = sys.argv[2]

    if action == 'add':
        add_blackhole(ip_address)
    elif action == 'remove':
        remove_blackhole(ip_address)
    else:
        print("Ação inválida. Use 'add' para adicionar ou 'remove' para remover.")
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
