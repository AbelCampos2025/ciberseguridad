# Laboratorio 01 — ARP, enrutamiento y DNS

## Objetivo

Interpretar la caché ARP, la tabla de rutas IPv4 y una consulta DNS en Windows.

## Entorno sanitizado

- Equipo Windows: `192.168.1.113/24`
- Gateway LAN: `192.168.1.1`
- Interfaz Hyper-V: `172.24.192.1/20`
- Las direcciones MAC se omitieron intencionalmente.

## Comandos observados

```powershell
arp -a
route print -4
nslookup example.com
```

## Hallazgos

1. La caché ARP contenía asociaciones dinámicas para el gateway, otro equipo de la LAN y una instancia conectada a la red virtual de WSL.
2. Las direcciones broadcast se asociaban con `ff-ff-ff-ff-ff-ff` y las multicast con el prefijo Ethernet `01-00-5e`.
3. La ruta `192.168.1.0/24` estaba en vínculo mediante la interfaz Wi-Fi.
4. La ruta `172.24.192.0/20` estaba en vínculo mediante Hyper-V.
5. La ruta predeterminada `0.0.0.0/0` enviaba el resto del tráfico al gateway `192.168.1.1`.
6. La consulta DNS respondió mediante un servidor con dirección IPv6 ULA y devolvió registros IPv4 (`A`) e IPv6 (`AAAA`).
7. El texto `Unknown` para el servidor DNS no indicaba un fallo: faltaba un nombre obtenido por resolución inversa.

## Ejercicios resueltos

| Destino | Ruta | Resultado |
|---|---|---|
| `192.168.1.218` | `192.168.1.0/24` | Directo por Wi-Fi |
| `172.24.198.76` | `172.24.192.0/20` | Directo por Hyper-V |
| `172.24.205.50` | `172.24.192.0/20` | Directo por Hyper-V |
| `172.24.210.10` | `0.0.0.0/0` | Mediante `192.168.1.1` |
| `8.8.8.8` | `0.0.0.0/0` | Mediante `192.168.1.1` |

## Conclusión

Una dirección se entrega directamente cuando coincide con una red conectada. Si no existe una ruta más específica, Windows usa la ruta predeterminada. Para una entrega local IPv4 necesita resolver la dirección MAC mediante ARP.

## Seguridad

ARP carece de autenticación fuerte y puede ser manipulado. No se realizaron pruebas ofensivas y cualquier futura demostración de ARP spoofing se limitará a un laboratorio aislado y autorizado.
