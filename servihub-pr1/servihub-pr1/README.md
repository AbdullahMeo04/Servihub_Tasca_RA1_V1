# PR1 · ServiHub v1 — Llançador multiprocés

- **Alumne:** Abdullah Meo
- **Curs:** DAM2 · 2026-2027
- **Mòdul:** 0490 · Programació de serveis i processos
- **Professor:** Ignasi López Aylagas
- **Activitat:** PR1 · AEA1 — Programació multiprocés

## Descripció general del projecte

- **Objectiu:** Desenvolupar **ServiHub v1**, un llançador multiprocés en Java 21 capaç d'executar programes externs com a processos fills i gestionar-ne tot el cicle de vida.
- **Tecnologies utilitzades:** API nativa de processos de Java (`ProcessBuilder`, `Process` i `ProcessHandle`), fils d'execució concurrents per evitar bloquejos de fluxos (`FilLector`) i proves unitàries automatitzades amb **JUnit 5**.
- **Capacitats principals:**
    - Llançament síncron i asíncron de processos fills.
    - Captura independent de la sortida estàndard (`stdout`) i d'errors (`stderr`).
    - Comunicació bidireccional amb el fill mitjançant variables d'entorn, entrada estàndard (`stdin`) i canonades (*pipelines*).
    - Control de temps màxim d'execució (*timeouts*) amb finalització amable i forçada.
    - Redirecció de sortides a fitxers i generació d'un informe final amb `PID`, durada, codi de retorn i estat de cada feina.

## Desglossament dels 20 forats implementats (Blocs A – K)

- **Bloc A · Llançar un procés i esperar-lo (`Llancador.executa`)**
    - **Forat 01 (`pb.inheritIO()`):** Configura el procés fill perquè comparteixi l'entrada, sortida i error estàndard directament amb la consola del procés pare.
    - **Forat 02 (`Process p = pb.start()`):** Inicia l'execució del procés fill i en desa la referència.
    - **Forat 03 (`return p.waitFor()`):** Bloqueja el pare fins que el fill finalitza i retorna el seu codi de sortida.

- **Bloc B · Llegir la sortida d'un procés (`Llancador.llegeixTot`)**
    - **Forat 04 (`while ((linia = r.readLine()) != null)`):** Llegeix un `InputStream` línia a línia amb `BufferedReader` fins a detectar el final del flux (`EOF` / `null`) i les uneix amb salts de línia.

- **Bloc C · Classificació del codi de retorn (`Estat.segonsCodi`)**
    - **Forat 05 (`return codi == 0 ? CORRECTA : FALLIDA`):** Avalua el codi de sortida del procés a `Estat.java` per determinar si la feina ha acabat correctament (`0`) o ha fallat (qualsevol altre valor).

- **Bloc D · Captura separada de sortida i error (`Llancador.executaICaptura`)**
    - **Forat 06 (`return Resultat.noArrencada(nom, e.getMessage())`):** Captura l'excepció `IOException` si el programa no existeix o no es pot arrencar i retorna un resultat controlat.
    - **Forat 07 (`FilLector.engega(p.getErrorStream())`):** Inicia un segon fil lector concurrent per buidar el canal d'errors (`stderr`) en paral·lel a `stdout` i evitar interbloquejos (*deadlocks*).
    - **Forat 08 (`long pid = p.pid()`):** Obté l'identificador únic del procés fill assignat pel sistema operatiu.

- **Bloc E · Variables d'entorn (`Llancador.executaAmbEntorn`)**
    - **Forat 09 (`pb.environment().putAll(entorn)`):** Afegeix totes les parelles clau-valor del mapa d'entorn al `ProcessBuilder` abans d'iniciar el fill.

- **Bloc F · Escriptura a l'entrada estàndard (`Llancador.enviaPerEntrada`)**
    - **Forat 10 (`p.getOutputStream()`):** Obté el flux de sortida del pare connectat a l'entrada estàndard (`stdin`) del procés fill per enviar-li dades.
    - **Forat 11 (`capAlFill.close()`):** Tanca el `PrintWriter` un cop enviades totes les línies perquè el fill rebi el senyal `EOF` i no quedi esperant indefinidament.

- **Bloc G · Control de temps màxim (`Llancador.executaAmbTempsMaxim`)**
    - **Forat 12 (`p.waitFor(segons, TimeUnit.SECONDS)`):** Espera la finalització del fill fins a un límit de temps determinat.
    - **Forat 13 (`p.destroy()`):** Si s'excedeix el temps màxim, envia una petició de finalització amable al procés fill.
    - **Forat 14 (`p.destroyForcibly()`):** Si passats 2 segons addicionals el fill continua actiu, el força a aturar-se immediatament i marca la feina com a `AVORTADA`.

- **Bloc H · Execució de feines en paral·lel (`Llancador.executaEnParallel`)**
    - **Forat 15 (`fills.add(new ProcessBuilder(f.ordre()).inheritIO().start())`):** Llança totes les feines de la llista de manera concurrent sense esperar-les dins del primer bucle.
    - **Forat 16 (`int codi = p.waitFor()`):** Espera seqüencialment la finalització de cada procés ja en marxa per recollir-ne el codi de retorn i la durada total.

- **Bloc I · Redirecció de sortida i error a fitxers (`Llancador.executaAFitxers`)**
    - **Forat 17 (`pb.directory(carpeta.toFile())`):** Estableix la carpeta indicada com a directori de treball del procés fill.
    - **Forat 18 (`pb.redirectOutput(carpeta.resolve(nom + ".out").toFile())`):** Redirigeix la sortida estàndard del fill al fitxer `<nom>.out` dins la carpeta de destinació.

- **Bloc J · Canonada entre processos (`Llancador.canonada`)**
    - **Forat 19 (`ProcessBuilder.startPipeline(builders)`):** Llança totes les etapes de manera simultània connectant la sortida estàndard de cada procés amb l'entrada estàndard del següent (`A | B | C`).

- **Bloc K · Identificació del procés pare (`Llancador.pidDeServiHub`)**
    - **Forat 20 (`return ProcessHandle.current().pid()`):** Consulta i retorna el `PID` del procés actual de la JVM on s'executa ServiHub.

## Execució i verificació

- **Proves unitàries (JUnit 5):**
    - Des d'IntelliJ IDEA: executar la classe `LlancadorTest` per verificar que els **25 tests** passen en verd.
    - Des de terminal (amb Maven instal·lat):
      ```bash
      mvn test
      ```
- **Execució de l'aplicació principal (`App`):**
    - Executa una demostració completa de tots els blocs i imprimeix l'informe final de feines:
      ```bash
      mvn compile exec:java -Dexec.mainClass="cat.pratfp.servihub.pr1.App"
      ```