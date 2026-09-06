# Cheatsheet de networking

## Valores del laboratorio

- LAN: `192.168.1.0/24`
- Gateway: `192.168.1.1`
- Red virtual WSL/Hyper-V: `172.24.192.0/20`
- Gateway virtual observado: `172.24.192.1`

## Diagnóstico en Windows

```powershell
ipconfig /all
Get-NetIPConfiguration
Get-NetRoute
ping 127.0.0.1
ping 192.168.1.1
Resolve-DnsName example.com
Test-NetConnection example.com -Port 443
tracert example.com
arp -a
```

No publiques la salida sin eliminar nombres, IP públicas y otros datos identificables.
