# AdGuard Home API

## 💡 Szybki Start w Postmanie

1. **New Request** → **GET**
2. **URL**: `http://192.168.1.222:3000/control/status`
3. **Authorization** → **Basic Auth**
   - Username: `mateusz`
   - Password: `[wpisz hasło]`
4. **Send**

   Większość endpointów znajduje się w `/control/...`

---

## 🚀 Najważniejsze Endpointy

### 1. Status Serwera

```
GET http://192.168.1.222:3000/control/status
```

Zwraca - co nas interesuje:

- status ochrony (enabled/disabled)
- czas do włączenia ochrony (jeśli jest wyłączona)
- status czy AdGuard Home jest aktualnie włączony

### 2. Statystyka

```
GET http://192.168.1.222:3000/control/stats
```

Zwraca:

- `num_dns_queries` - **całkowita liczba zapytań**
- `num_blocked_filtering` - **całkowita liczba zablokowanych**
- `top_queried_domains` - najczęściej odpytywane domeny
- `top_clients` - **top klientów (ale tylko IP, bez nazw!)**
- `top_blocked_domains` - top zablokowanych domen
- `avg_processing_time` - średni czas odpowiedzi

**Problem**: `top_clients` zwraca tylko IP adresy, bez nazw.
**Rozwiązanie**: Musisz połączyć z endpointem `/clients` aby uzyskać nazwy! - opis w sekcji "🎯 TOP Clientów z Nazwami"

### 3. Ochrona (On/Off)

Uwaga, bo czas podaje się w milisekundach, więc 30 minut to 1800000 ms.

```
POST http://192.168.1.222:3000/control/protection

Body (JSON):
{
  "enabled": false,
  "duration": 1800000
}
```

---

## Dodatkowe Endpointy

### 1. Sprawdzenie dostępnych aktualizacji

```
POST http://192.168.1.222:3000/control/version.json

Body (JSON):
{
  "recheck_now": true
}
```

**Odpowiedź:**

```json
{
  "disabled": false,
  "new_version": "v0.107.72",
  "announcement": "AdGuard Home v0.107.72 is now available!",
  "announcement_url": "https://github.com/AdguardTeam/AdGuardHome/releases/tag/v0.107.72",
  "can_autoupdate": false
}
```

### ⚠️ Jak sprawdzić czy UPDATE jest DOSTĘPNY?

**Problem**: Po updacie `new_version` wciąż pokazuje tę samą wersję!

**Rozwiązanie**: Musisz **porównać z aktualną wersją**:

1. **Pobierz aktualną wersję z `/status`:**

```
GET http://192.168.1.222:3000/control/status
```

```json
{
  "version": "v0.107.72",
  "running": true,
  ...
}
```

2. **Porównaj wersje:**

```javascript
const statusResponse = await fetch("/control/status");
const versionResponse = await fetch("/control/version.json", {
  method: "POST",
});

const { version } = await statusResponse.json();
const { new_version } = await versionResponse.json();

const hasUpdate = version !== new_version;

if (hasUpdate && can_autoupdate) {
  console.log(`Update dostępny: ${version} → ${new_version}`);
  // Pokaż przycisk "Zaktualizuj"
} else if (hasUpdate && !can_autoupdate) {
  console.log(`Update dostępny, ale nie można auto-update: ${new_version}`);
  // Pokaż info że trzeba ręcznie
} else {
  console.log("Masz najnowszą wersję!");
}
```

**Czyli:**

- `version` (z `/status`) = aktualna zainstalowana wersja
- `new_version` (z `/version.json`) = dostępna do pobrania wersja
- **Jeśli są równe = masz najnowszą wersję** ✅

I tutaj chcielibyśmy żeby UI np. pokazywało zielony "Aktualny" lub czerwony "Dostępny update!" w zależności od tego czy `version === new_version` czy nie.

### 2. Puszczanie aktualizacji (Auto-Update)

```
POST http://192.168.1.222:3000/control/update
```

**Odpowiedź: 200 OK**

Uruchamia automatyczną aktualizację AdGuard Home.

**⚠️ Uwaga:**

- Server będzie niedostępny przez kilka minut podczas updatu
- Po updacie service restartuje się automatycznie
- Możliwy jest update tylko jeśli `can_autoupdate: true`
- Przycisk Update powinien być dostępny tylko gdy `version !== new_version` i `can_autoupdate: true`

---

## 🎯 TOP Clientów z Nazwami (rDNS/Niestandardowe)

### Krok 1: Pobierz listę klientów z nazwami

```
GET http://192.168.1.222:3000/control/clients
```

**Odpowiedź zawiera:**

```json
{
  "clients": [
    {
      "name": "PC - Mateusz",
      "ids": ["192.168.1.100"]
    },
    {
      "name": "Phone - Anna",
      "ids": ["192.168.1.50"]
    }
  ],
  "auto_clients": [
    {
      "ip": "192.168.1.200",
      "name": "printer.local",
      "source": "rDNS"
    }
  ]
}
```

**Wyjaśnienie:**

- `clients` - ręcznie przypisane nazwy
- `auto_clients` - automatyczne nazwy z rDNS (reverse DNS)

### Krok 2: Pobierz TOP clientów ze statystyki

```
GET http://192.168.1.222:3000/control/stats
```

**Odpowiedź (interesująca nas część):**

```json
{
  "top_clients": [
    { "192.168.1.100": 150 },
    { "192.168.1.50": 120 },
    { "192.168.1.200": 85 }
  ],
  "num_dns_queries": 2500,
  "num_blocked_filtering": 320
}
```

### Krok 3: Połącz dane
