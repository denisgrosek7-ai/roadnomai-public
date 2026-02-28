# CI Hardening (Korak 3)

## Cilj

Doseči **3 zaporedne čiste prehode** za:
- `CI (main)`
- `CI (nightly golden)`

## Postopek

1. Potrdi, da je nastavljen `GOOGLE_MAPS_API_KEY` secret v GitHub repository secrets.
2. Pushni commit na `main`.
3. Zaženi `CI (nightly golden)` tudi ročno (`workflow_dispatch`) na istem SHA.
4. Ponovi korak 2+3, dokler nimaš 3 zaporednih uspešnih poskusov.

## Kaj šteje kot "clean pass"

- noben failed job v workflowu
- noben skipped job zaradi manjkajočega API key-a v `main/nightly` golden jobih
- brez flaky rerun-only pass (šteje prvi run na commit)

## Evidence Capture

Izpolni `docs/korak3/ci-evidence.md`:
- commit SHA
- datum/čas v `Europe/Ljubljana`
- URL do `main` workflow runa
- URL do `nightly` workflow runa
- rezultat (`PASS` / `FAIL`)

