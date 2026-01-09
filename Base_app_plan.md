Główne założenia aplikacji bazowej:

Główny UI:

1. Główny interfejs użytkownika, który umożliwia dostęp do różnych funkcji aplikacji. Dla każdego użyytkownika dostępny jest unikalny dashboard.
2. Konto admina przydziela dostępne moduły i funkcje dla poszczególnych użytkowników. Moduły mogą być dodawane lub usuwane w zależności od potrzeb.
3. Moduły są "bloczkami", które dodaje się do bazowego interfejsu użytkownika.
4. Uptime monitoring dla usług.
5. Podstawowe zarządzanie usługami (start, stop, restart).

UI do przeglądania logów usług.

1. UI do historii logów integracji z API.
2. UI do historii powiadomień.
3. UI do historii akcji użytkowników.

UI logowania:

1. Interfejs logowania pozwalający na zalogowanie się na różne konta użytkowników.
2. Rate limiting dla prób logowania.

UI zarządzania użytkownikami:

1. Interfejs do zarządzania użytkownikami, w tym tworzenie, edytowanie i usuwanie kont użytkowników.
2. Możliwość zmiany ról użytkowników (np. admin, użytkownik standardowy).
3. Możliwość resetowania haseł użytkowników.
4. Bulk operations - edycja wielu użytkowników naraz

UI zarządzania modułami:

1. Podstrona dla admina do zarządzania jakie moduły dostępne są dla poszczególnych użytkowników.
2. Strona pozwala adminowi, jak i użytkownikom końcowym na układanie ich własnego dashboardu poprzez dodawanie i usuwanie modułów.

UI do zarządzania wysyłaniem powiadomień:

1. Interfejs do konfiguracji powiadomień na discorda lub telegrama.
2. Interfejs do zarządzania które i jakie powiadomienia są wysyłane.
3. Customowe powiadomienia tworzone przez użytkownika.

UI do zarządzania integracjami z API:

1. Pozwala wybrać z presetów integracji z różnymi API.
2. Pozwala wybrać odpowiednie IP i porty dla integracji.
3. Pozwala dodać własne endpointy API.

Dodatkowe funkcje:

1. Export/import konfiguracji.
2. Rate limiting dla API.

Efekty "WOW" - czyli może się pojawi, ale nie wiem czy na pewno:

1. Responsywny design dostosowujący się do różnych rozdzielczości ekranów.
2. Ciemny i jasny motyw interfejsu użytkownika.
3. Animacje przy przejściach między podstronami i interakcjach z modułami.
4. Zmiana kolorów i motywów przez użytkownika.
5. Możliwość ustawiania własnego awatara użytkownika z predefiniowanych ikon.
6. Docker Compose template.
7. 2FA/MFA

Endpointy API do zaprogramowania w aplikacji bazowej:

1. Adguard Home
2. UniFi
3. TrueNAS CE
4. Proxmox VE

Backend aplikacji bazowej:

1. Autoryzacja i uwierzytelnianie użytkowników (logowanie, rejestracja, zarządzanie sesjami).
2. Zarządzanie użytkownikami (tworzenie, edytowanie, usuwanie kont użytkowników).
3. Zarządzanie modułami (dodawanie, usuwanie, konfigurowanie modułów dla użytkowników).
4. Zarządzanie powiadomieniami (konfiguracja i wysyłanie powiadomień).
5. Integracje z API (zarządzanie integracjami z różnymi API).
6. Przechowywanie ustawień użytkowników (motywy, układ dashboardu, preferencje powiadomień).
