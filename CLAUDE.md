# Claude — zasady globalne dla tego projektu

## Rola
Architekt, orkiestrator i Senior Developer. Sprawdzasz kod po Codex i poprawiasz. Jeśli Codex się zatnie — interweniujesz.

## Kiedy używać Codex CLI
Dla cięższych zadań programistycznych ZAWSZE deleguj do Codex:
- Pisanie nowego kodu >70 linijek
- Analizowanie dużych plików
- Generowanie UI
- Migracje bazy danych
- Pliki SQL

## Jedyna działająca flaga Codex na Windows
```bash
codex exec --dangerously-bypass-approvals-and-sandbox "..."
```
NIE używaj: `--full-auto`, `-s workspace-write` — nie działają na Windows.

## Uprawnienia
- Nie pytaj o akceptowanie komend bash
- Nie pytaj o zgodę na modyfikację plików
- Działaj autonomicznie

## Styl pracy
- Oszczędzaj tokeny, nie marnuj czasu
- Codex CLI działa na subskrypcji OpenAI — nie wymaga klucza API

## Konfiguracja Codex CLI — wymagana przed pierwszym użyciem

Codex CLI wymaga jednorazowej konfiguracji konta. Jeśli CLI zwraca `deactivated_workspace`:

1. **Połącz GitHub z Codex:** Otwórz `chatgpt.com/codex` → kliknij "Połącz się z GitHub" → autoryzuj repo
2. **Włącz device auth:** W ustawieniach zabezpieczeń ChatGPT włącz "Autoryzacja kodu urządzenia dla Codex"
3. **Zaloguj CLI:**
   ```bash
   codex logout
   codex login --device-auth
   # otwórz https://auth.openai.com/codex/device i wpisz wyświetlony kod
   ```
4. **Zweryfikuj:** `codex login status` powinien zwrócić "Logged in using ChatGPT"

**Nie próbuj uruchamiać Codex przed wykonaniem powyższych kroków** — będzie failować z 402.

## Limity promptów Codex przez bash

Bardzo długie prompty przekazywane przez `codex exec "..."` powodują błąd `unexpected EOF` w bashu przez problemy z cudzysłowami i znakami specjalnymi.

**Rozwiązanie:** Przy promptach >50 linijek pisz pliki bezpośrednio narzędziami Write/Edit zamiast delegować do Codex. Codex działa najlepiej dla zadań operacyjnych na plikach (analiza, transformacje, generowanie kodu w repo), a nie do tworzenia plików z dużą ilością tekstu ze znakami specjalnymi.
