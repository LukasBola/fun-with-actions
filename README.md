# Informacje o automatycznym uruchamianiu workflowów

W repozytorium możesz kontrolować, czy GitHub Actions (workflowy) uruchamiają się automatycznie. Poniżej są dwie proste opcje, które możesz zastosować, aby zatrzymać zautomatyzowane uruchomienia i oszczędzić darmowe minuty podczas wykonywania kolejnych lekcji.

## Opcja 1 — Zmień rozszerzenie pliku na `.yml.disabled`

Na chwilę obecną w repozytorium masz dwie konfiguracje – `blank.yml` oraz `pylint.yml`.
Aby tymczasowo wyłączyć ich automatyczne uruchamianie, zmień nazwy tych plików tak, aby ich rozszerzenia stały się `.yml.disabled`. Przykłady:

- `blank.yml` → `blank.yml.disabled`
- `pylint.yml` → `pylint.yml.disabled`

Dlaczego to działa:

- GitHub Actions rozpoznaje workflowy wyłącznie w katalogu `.github/workflows/` z rozszerzeniem `.yml` lub `.yaml`. Zmiana rozszerzenia na inne (np. `.yml.disabled`) powoduje, że plik nie jest już traktowany jako workflow i nie będzie uruchamiany.

Wady:

- Pliki przestaną być widoczne jako workflowy w interfejsie Actions. Przywrócenie działania wymaga ponownego przywrócenia rozszerzenia `.yml` i wypchnięcia zmian.

## Opcja 2 — Zignoruj wszystkie pushy (branches-ignore)

Możesz też wyłączyć uruchamianie workflowów na zdarzenia `push` przez dodanie (lub modyfikację) sekcji `on` w plikach workflow tak, aby pushy były ignorowane:

```yaml
on:
  push:
    branches-ignore:
      - "**"
```

Dlaczego to działa:

- Ta konfiguracja pozostawia workflow w repozytorium, ale powoduje, że żadne zdarzenie `push` nie uruchomi tego workflowa. Nadal możesz uruchomić workflow ręcznie z zakładki Actions (workflow_dispatch) lub pozostawić inne wyzwalacze (np. schedule).

Wady:

- Jeśli workflow ma być całkowicie wyłączony (np. również przed harmonogramem), trzeba dodatkowo usunąć lub zmodyfikować sekcję `schedule` albo wyłączyć workflow przez zmianę rozszerzenia pliku.

## Szybkie komendy (przykład)

Zmiana rozszerzenia pliku:

```bash
git mv .github/workflows/blank.yml .github/workflows/blank.yml.disabled
git mv .github/workflows/pylint.yml .github/workflows/pylint.yml.disabled
git commit -m "Disable workflows during lessons: rename to .yml.disabled"
git push origin main
```

Dodanie `branches-ignore` do istniejącego pliku (edytuj plik, a potem commit i push):

```bash
# edytuj plik .github/workflows/your-workflow.yml i dodaj/zmodyfikuj sekcję `on`
git add .github/workflows/your-workflow.yml
git commit -m "Disable push triggers for workflow"
git push origin main
```

## Zależności między jobami — `needs`

Możesz ustawić kolejność wykonywania jobów w workflow za pomocą pola `needs`. Dzięki temu job B uruchomi się dopiero po pomyślnym zakończeniu jobu A.

Przykład:

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - run: echo setup

  test:
    needs: setup
    runs-on: ubuntu-latest
    steps:
      - run: echo test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: echo deploy
```

Kilka uwag:

- Job z `needs` uruchomi się tylko jeśli wskazane joby zakończyły się sukcesem.
- Możesz podać wiele zależności: `needs: [setup, prepare]`.
- `needs` pozwala odwołać się do outputs jobów (np. `needs.setup.outputs.myvalue`).
