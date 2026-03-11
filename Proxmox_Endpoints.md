# Proxmox VE API

## 💡 Szybki Start w Postmanie

### API Token (zalecana i jedyna opcja)

Token tworzysz w GUI Proxmoxa:

`Datacenter -> API Tokens -> Add`

Przy dodawaniu podajesz tylko:
- user (`<USER>@<REALM>`),
- `Token name`.

Po zapisaniu dostajesz `Token value`.

1. **New Request** → **GET**
2. **URL**: `https://<IP_PROXMOX>:8006/api2/json/version`
3. W zakładce **Authorization** dodaj Value
5. **Send**

## 🔐 Najważniejsze info o połączeniu do naszej apki

- Base URL Proxmoxa:

```text
https://<IP_PROXMOX>:8006/api2/json
```

- Odpowiedzi API są zwykle w formacie:

```json
{
  "data": ...
}
```

- Oficjalne źródło endpointów: Proxmox API Viewer
  - https://pve.proxmox.com/pve-docs/api-viewer/index.html

---

## 🚀 Najważniejsze Endpointy (monitoring + podstawowe zarządzanie)

### 1) Status węzłów (serwerów Proxmox)

#### Lista węzłów

```http
GET /nodes
```

Przykład pełnego URL:

```text
https://<IP_PROXMOX>:8006/api2/json/nodes
```

#### Status konkretnego węzła

```http
GET /nodes/{node}/status
```

Zwraca m.in.:
- CPU (`cpu`)
- RAM (`memory.used`, `memory.total`)
- Uptime (`uptime`)
- `loadavg`
- `rootfs` usage
- memory

---

### 2) Szybki przegląd całego klastra (bardzo przydatne do dashboardu)

```http
GET /cluster/resources
```

Można filtrować typ:

```http
GET /cluster/resources?type=node
GET /cluster/resources?type=vm
GET /cluster/resources?type=storage
```

To zwykle najlepszy endpoint na ekran "overview", bo zwraca w jednym miejscu node + VM + CT (LXC) z podstawowym stanem.

TU MOGLIBYŚMY ZROBIĆ TAKIE COŚ, ŻE MOŻESZ SOBIE MONITOROWAĆ KONKRETNE VM/LXC CZY SĄ URUCHOMIONE ITD. JAKO OSOBNE KAFELKI W WEBOWYM UI

I TAK SAMO OCZYWIŚCIE TO MOŻNA WYKORZYSTAĆ DO MONITOROWANIA KONKRETNYCH NODE'ÓW (CO WYKORZYSTAMY TO NASZ WYBÓR, PROXMOX DAJE DUŻO OPCJI)

---

### 3) VM (QEMU) – status i szczegóły

#### Lista VM na node

```http
GET /nodes/{node}/qemu
```

#### Aktualny status konkretnej VM

```http
GET /nodes/{node}/qemu/{vmid}/status/current
```

Przydatne pola:
- `status` (`running` / `stopped`)
- `cpu`
- `mem`, `maxmem`
- `uptime`
- `name`

---

### 4) Kontenery LXC – status i szczegóły

#### Lista kontenerów na node

```http
GET /nodes/{node}/lxc
```

#### Aktualny status konkretnego kontenera

```http
GET /nodes/{node}/lxc/{vmid}/status/current
```

Przydatne pola:
- `status` (`running` / `stopped`)
- `cpu`
- `mem`, `maxmem`
- `uptime`
- `name`

---

### 5) Akcje start/stop/restart (VM i LXC)

#### VM (QEMU)

##### Start VM

```http
POST /nodes/{node}/qemu/{vmid}/status/start
```

##### Stop VM (twardy)

```http
POST /nodes/{node}/qemu/{vmid}/status/stop
```

##### Shutdown VM (grzeczny ACPI)

```http
POST /nodes/{node}/qemu/{vmid}/status/shutdown
```

##### Reboot VM

```http
POST /nodes/{node}/qemu/{vmid}/status/reboot
```

#### LXC

##### Start kontenera

```http
POST /nodes/{node}/lxc/{vmid}/status/start
```

##### Stop kontenera (twardy)

```http
POST /nodes/{node}/lxc/{vmid}/status/stop
```

##### Shutdown kontenera (grzeczny)

```http
POST /nodes/{node}/lxc/{vmid}/status/shutdown
```

##### Reboot kontenera

```http
POST /nodes/{node}/lxc/{vmid}/status/reboot
```

> Większość akcji zwraca `UPID` (ID taska), a nie natychmiast finalny stan.

---

### 6) Sprawdzanie tasków po akcji (ważne do UI)

Po `start/stop/reboot` dostajesz np.:

```json
{ "data": "UPID:node:..." }
```

#### Status taska

```http
GET /nodes/{node}/tasks/{upid}/status
```

#### Log taska

```http
GET /nodes/{node}/tasks/{upid}/log
```

UI może działać tak:
1. Wyślij akcję (np. `start`).
2. Zapisz `UPID`.
3. Polling co 1–2 sekundy na `/tasks/{upid}/status`.
4. Po zakończeniu odśwież `/status/current` VM/CT.

---

### 7) Dodatkowe endpointy warte rozważenia

#### Wersja Proxmox

```http
GET /version
```

#### Krótkie informacje o klastrze

```http
GET /cluster/status
```

#### Metryki czasowe (wykresy CPU/RAM/NET) – node

```http
GET /nodes/{node}/rrddata
```

#### Metryki czasowe – VM

```http
GET /nodes/{node}/qemu/{vmid}/rrddata
```

#### Metryki czasowe – LXC

```http
GET /nodes/{node}/lxc/{vmid}/rrddata
```

---

## ✅ Minimalny zestaw endpointów „must have” do HomeLabUI

Jeśli chcemy szybko dowieźć monitoring + podstawowe zarządzanie, wystarczy:

1. `GET /cluster/resources`
2. `GET /nodes`
3. `GET /nodes/{node}/status`
4. `GET /nodes/{node}/qemu/{vmid}/status/current`
5. `GET /nodes/{node}/lxc/{vmid}/status/current`
6. `POST /nodes/{node}/qemu/{vmid}/status/start|stop|reboot|shutdown`
7. `POST /nodes/{node}/lxc/{vmid}/status/start|stop|reboot|shutdown`
8. `GET /nodes/{node}/tasks/{upid}/status`

To pokrywa:
- status hostów,
- status VM i kontenerów,
- start/restart/stop,
- feedback o wykonaniu akcji.

---

## Źródła (sprawdzone)

- Proxmox VE API (official): https://pve.proxmox.com/wiki/Proxmox_VE_API
- Proxmox API Viewer (official): https://pve.proxmox.com/pve-docs/api-viewer/index.html
