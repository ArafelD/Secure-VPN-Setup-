

# 🔐 Secure VPN Setup (Linux & Windows)

![Secure VPN Setup Logo](https://private-us-east-1.manuscdn.com/sessionFile/lKW00ksl2DprHV1bJ8odUZ/sandbox/uNgWvMGB6mrY6yuxuaS0wu-images_1773871802022_na1fn_L2hvbWUvdWJ1bnR1L3Byb2plY3RfbG9nbw.png?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvbEtXMDBrc2wyRHBySFYxYko4b2RVWi9zYW5kYm94L3VOZ1d2TUdCNm1yWTZ5dXh1YVMwd3UtaW1hZ2VzXzE3NzM4NzE4MDIwMjJfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwzQnliMnBsWTNSZmJHOW5idy5wbmciLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE3OTg3NjE2MDB9fX1dfQ__&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=e-ngSvL8sNVhEFnpOKDVTqQo5Qqr-r4fLbCswb50vHnZUfVxnrX5Z0EBNm4m31E5XiwgX0UKZjc1uV5cz3HRYSeTydY4TGhb5OgvExHR369kBpnu3uVnXwH8SolHq7oQbqwwe1iI7X6~3uK7tUMQ~fbmjJRkhF1KI9B9QG5c1ZWuROnjjpq3VDhClReEVeT3S3H4sIhR5pjki2YcRgGOGjGaKqRRzmaR7X1cofsQa8TAEKEO3kzfC7-KUe2sPtsDc0i0u-pXZo5EiSsOVCDLILKYBOTIxJIjXUF-kMfeG7eWReZ--WBC9pm0llEgKcCuapcJPrBl7Om59kA8h1sW3g__)

![Linux](https://img.shields.io/badge/Linux-Ubuntu%2FDebian-blue?logo=linux)
![Windows](https://img.shields.io/badge/Windows-11-blue?logo=windows)
![Security](https://img.shields.io/badge/Security-Hardened-green)
![Firewall](https://img.shields.io/badge/Firewall-UFW%20%7C%20Windows%20Defender-orange)
![VPN](https://img.shields.io/badge/VPN-ProtonVPN-purple)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## 📌 Visão Geral

Este projeto demonstra a implementação de um ambiente seguro em **Linux e Windows**, combinando:

- 🔐 VPN
- 🛡️ Firewall restritivo
- 🌐 DNS seguro
- 🚫 Kill Switch real

👉 Objetivo: garantir que **todo o tráfego passe exclusivamente pelo túnel VPN**, bloqueando qualquer vazamento.

---

## 🚀 Funcionalidades

- ✔️ VPN multiplataforma (ProtonVPN, OpenVPN, WireGuard)
- ✔️ Kill Switch robusto
- ✔️ Proteção contra vazamento de DNS
- ✔️ Firewall com política *deny by default*
- ✔️ Configuração prática e replicável

---


---

# 🐧 Linux (Ubuntu / Debian / Zorin)

## 1. Instalação da VPN

```bash
curl -fsSL https://repo.protonvpn.com/debian/public_key.asc | gpg --dearmor | sudo tee /usr/share/keyrings/protonvpn.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/protonvpn.gpg] https://repo.protonvpn.com/debian stable main" | sudo tee /etc/apt/sources.list.d/protonvpn.list

sudo apt update
sudo apt install proton-vpn-gnome-desktop
```

## 2. Firewall + Kill Switch

```bash
sudo ufw default deny outgoing
sudo ufw default deny incoming

sudo ufw allow in on lo
sudo ufw allow out on lo

sudo ufw allow out 53
sudo ufw allow out 80/tcp
sudo ufw allow out 443/tcp
sudo ufw allow out 1194/udp
sudo ufw allow out 51820/udp

sudo ufw deny out on eth0
sudo ufw allow out on tun0

sudo ufw enable
```

## 3. DNS Seguro

```bash
sudo nano /etc/systemd/resolved.conf
```

Adicione/modifique:

```ini
DNS=1.1.1.1 1.0.0.1
DNSOverTLS=yes
```

Reinicie:

```bash
sudo systemctl restart systemd-resolved
```

---

# 🪟 Windows

## 1. Instalação da VPN

Instale o cliente de sua preferência (Proton VPN, OpenVPN, WireGuard, etc.). Certifique-se de que a interface virtual da VPN seja identificada (geralmente aparece como "ProtonVPN" ou "WireGuard" nas conexões de rede).

## 2. Firewall + Kill Switch

Utilize o PowerShell como Administrador para criar regras que bloqueiam o tráfego fora da VPN.

**Passo A: Bloquear todo o tráfego de saída por padrão (Perfil Público/Privado)**

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -DefaultOutboundAction Block
```

**Passo B: Permitir tráfego apenas para o IP do servidor da sua VPN**

*(Substitua `IP_DO_SERVIDOR` pelo IP real do servidor VPN que você utiliza)*

```powershell
New-NetFirewallRule -DisplayName "Allow VPN Server" -Direction Outbound -RemoteAddress IP_DO_SERVIDOR -Action Allow
```

**Passo C: Permitir tráfego na interface da VPN**

```powershell
# Identifique o nome da interface da sua VPN
Get-NetAdapter
# Permita tráfego nela (Exemplo: interface chamada "ProtonVPN")
New-NetFirewallRule -DisplayName "Allow VPN Tunnel" -Direction Outbound -InterfaceAlias "ProtonVPN" -Action Allow
```

## 3. DNS Seguro

No Windows 11, você pode habilitar o DoH via PowerShell:

```powershell
# Habilitar DoH globalmente
reg add "HKLM\System\CurrentControlSet\Services\Dnscache\Parameters" /v "EnableAutoDoh" /t REG_DWORD /d "2" /f

# Configurar servidores DNS (Cloudflare)
Set-DNSClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses ("1.1.1.1", "1.0.0.1")
```

---

## 🧪 Testes de Validação (Ambas as Plataformas)

1.  **Teste de IP:** `curl ifconfig.me` (Deve mostrar o IP da VPN).
2.  **Teste de Kill Switch:** Desconecte a VPN. O comando `curl ifconfig.me` deve falhar ou ficar "preso", indicando que não há vazamento pelo IP real.
3.  **Teste de DNS:** Utilize sites como [dnsleaktest.com](https://dnsleaktest.com) para garantir que apenas os servidores configurados apareçam.

---

## 🧠 Arquitetura



## 🚧 Problemas Reais Enfrentados

| Problema                 | Causa                                     | Solução                                        |
| :----------------------- | :---------------------------------------- | :--------------------------------------------- |
| `Unable to locate package` | Repositório não configurado               | Adição do repositório                          |
| `apt-secure error`       | Falta de GPG                              | Configuração de keyring                        |
| Sem internet             | Firewall bloqueando tudo                  | Liberação de portas                            |
| VPN não conecta          | UFW bloqueando saída                      | Permitir portas VPN                            |
| DNS não resolve          | Porta 53 bloqueada                        | Liberar DNS                                    |

---

## 🔐 Segurança Implementada

- ✔️ Bloqueio total por padrão
- ✔️ Tráfego permitido apenas via VPN
- ✔️ DNS protegido
- ✔️ Kill switch funcional

---

## 📊 Conclusão

Este projeto demonstra na prática conceitos de:

- 🔐 Segurança da Informação
- 🌐 Redes
- 🖥️ Administração Linux
- ⚙️ Infraestrutura

---

## 👨‍💻 Autor

Projeto desenvolvido como prática em Segurança da Informação, com foco em proteção de tráfego e privacidade digital.

---

## ⭐ Contribuição

Sinta-se livre para abrir PRs ou sugerir melhorias.
