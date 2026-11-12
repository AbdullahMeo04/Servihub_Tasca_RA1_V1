# PR1 · ServiHub v1 — Llançador multiprocés

- **Alumne:** Abdullah Meo
- **Curs:** DAM2 · 2026-2027
- **Mòdul:** 0490 · Programació de serveis i processos
- **Professor:** Ignasi López Aylagas
- **Activitat:** PR1 · AEA1 — Programació multiprocés

## Descripció

**ServiHub v1** és un llançador multiprocés en Java 21 (`ProcessBuilder`, `Process` i `ProcessHandle`) que executa processos fills, en captura la sortida i l'error per separat, gestiona variables d'entorn, envia dades per entrada estàndard, controla temps màxims d'execució, llança tasques en paral·lel, redirigeix sortides a fitxers i encadena processos mitjançant canonades.

## Blocs implementats

- **Bloc A (Forats 01–03):** Llançament de processos amb `inheritIO()` i espera de codi de retorn (`waitFor()`).
- **Bloc B (Forat 04):** Lectura de fluxos de sortida línia a línia fins a EOF (`BufferedReader`).
- **Bloc C (Forat 05):** Assignació d'estat (`CORRECTA` / `FALLIDA`) segons el codi de retorn a `Estat.java`.
- **Bloc D (Forats 06–08):** Captura concurrent de `stdout` i `stderr` amb `FilLector`, gestió d'errors d'arrencada i obtenció del `PID`.
- **Bloc E (Forat 09):** Configuració de variables d'entorn del procés fill (`pb.environment().putAll(...)`).
- **Bloc F (Forats 10–11):** Escriptura a l'entrada estàndard del fill (`getOutputStream()`) i tancament del flux per enviar EOF.
- **Bloc G (Forats 12–14):** Control de temps màxim amb `waitFor(segons, TimeUnit.SECONDS)`, `destroy()` i `destroyForcibly()`.
- **Bloc H (Forats 15–16):** Execució concurrent de múltiples feines en paral·lel.
- **Bloc I (Forats 17–18):** Directori de treball i redirecció de `stdout` i `stderr` a fitxers `.out` i `.err`.
- **Bloc J (Forat 19):** Canonada entre processos amb `ProcessBuilder.startPipeline(...)`.
- **Bloc K (Forat 20):** Obtenció del `PID` del procés actual amb `ProcessHandle.current().pid()`.

## Execució i proves

```bash
# Executar els 25 tests JUnit 5
mvn test

# Executar l'aplicació principal
mvn compile exec:java -Dexec.mainClass="cat.pratfp.servihub.pr1.App"
```