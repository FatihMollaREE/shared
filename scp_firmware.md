
# Ziel: SCP-Firmware auf dem V4M

## Überblick
- **SCP-Firmware:** Dient zur Steuerung von Systemressourcen durch einen anderen Prozessor als den Application-Core, um diesen zu entlasten und die Effizienz zu steigern.
- **V4M-SoC:** 
  - Basierend auf einem Quad Cortex-A76 Core (2 Cores x 2 Cluster) und 3 Cortex-R52 Cores.
  - Verfügt über verschiedene Speicher: 2x QSPI (1x HyperFlash) und 1x eMMC. Diese speichern Firmware, die von den CPUs ausgeführt wird
    - eMMC: Speichert z. B. das Linux-Image und Device Tree Blob (dtb) da es günstiger ist, nachteil etwas langsamer als QSPI und Hyperflash
    - Programme werden zur Laufzeit in den RAM geladen und von der CPU ausgeführt
  - **RAM-Typen:** RT-VRAM (SRAM), System-RAM und DRAM (langsamer, aber günstiger). Adresszuordnungen können Address-Map (.xlsx) entnommen werden

---

## V4M Grayhawk Aufsetzen
1. **R-Car SDK installieren:** 
   - Sammlung von Ressourcen, Tools, Bibliotheken und Dokumentationen zur Erleichterung der Inbetriebnahme.
2. **Bootloader flashen:**
   - Anleitung im *Startup Guide* (S. 91).
   - **SCIF-Download-Mode:** Wird durch Switches aktiviert, um Images auf entsprechende Adressen zu laden (z. B. via Tera Term Macro).
   - Wechsel zu *Serial Flash ROM Boot Mode* nach dem Flashen.
   - Flash-Orte im *Startup Guide* (S. 63):  
     - **QSPI:** z. B. `EB20_0000` (RT-VRAM0).    \nur halb richtig
     - **eMMC:** z. B. `5000_0000` für U-Boot.    / genaueres im manual für den v4h 31.boot (erst später relevant)
3. **Linux-Image und dtb. flashen:** 
   - Linux image und dtb. wurden mithilfe von u-Boot aufs eMMC geflashed (Anleitung auf confluence/teams chat mit Matthias), dadurch kann nun auf dem A-76 linux laufen. U-Boot  ist hierbei eine Art bootloader, der einem unter anderem beim flashen und initializieren hilft

---

## SCP-Firmware Notes
### Wichtige Begriffe:
- **Protocols:** definieren einzelne Gruppen von System Control und Management messages, protocol specifications beschreiben die messages die sie supporten
- **Agents:** bezeichnet den Caller des SCMI
- **Platform:** beschreibt die Hardware, die die messages interpretieren und die notwendingen Funktionen bereitstellen
- **Resource:** beschreibt eine Komponente der Hardware-platform die mittels SCMI messages kontrolliert werden kann
- **Transport:** beschreibt die Methode mittels welcher Protocol messages zwischen Agent und Platform kommuniziert werden
- **Modules:** beschreibt einen Code-Block der eine bestimmte Operation oder Gruppe von Operationen ausführt,modul aufbau: z.118 framework.md und z.147 framework.md, config in z.171 framework.md
- **Elements:** Resource die von einem Modul besessen und bestimmt/kontrolliert wird, z.217 framework.md
- Indices and Identifiers:
   Indices:
   -  identifizieren items innerhalb "parent-scope", z.b ein element, event, notification, API innerhalb modul Kontext, oder module im Kontext der firmware --> indices werden  vom build system für jede firmware generiert und in fwk_module_idx.h platziert
   Identifier:
   -  identifieren items im globalen Kontext, so z.b das element eines Moduls von einem anderen Modul-Kontext,
   -  identifier haben typen die beschreiben welche informationen diese beinhalten, avalaible identifier: z.287 framework.md
- **APIs:** interfacing von modulen durch apis, declaring and using apis: z.308 ff framework.md
- **Events** strukturierte messages von einem source zu einem target, (module, element, sub-element) --> eventhandling z.357 framework.md
- **Notifications:** im Prinzip ein besonderes Event, wo ein modul eine Änderung in seinem Zustand mitteilt


### Produktimplementierung:
Produkt implementieren:
   ->product.mk file 
   ->ordner struktur siehe z.50 framework.md
   ->in jedem produkt muss eine firmware beinhalten, diese representiert ein software-image welches als teil des produkts geBUILDded wird.
      jede firmware muss hierbei die von ihr genutzten module aufzählen und eine config für diese beinhhalten
   ->jede firmware muss linker informationen beinhalten: -> fmw_memory.h z.69 framework.md
   ->in Toolchain-*.cmake architecture target fürs image beschreiben z.84 framework.md
			
---

## SCP-Firmware Ablauf / Funktionsweise notes:	
-> SCP Firmware funktioniert im Prinzip so:
   1. pre-runtime phase: Init, -> modules, elements, sub elements,..  z.426 framework.md
         - Module initialization
         - Element initialization
         - Post-initialization
         - Bind
         - Start
      
   2. runtime phase: 
      event loop waiting for event or interrupt, wenn ein event kommt dann wird dieses gehandelt, danach wird kontrolle wieder an main loop gegeben, framework wird genutzt um diese interaktion von modulen bereitzustellen und zu überprüfen
         (wenn pending events list leer ist, kann man zu 1. zurück kehren, (zb. um zu binden, das muss aber vorher festgelegt werden) )
   -> error handling ist architecture spezifisch, muss ich selber festlegen

   - module aufbau: fwk_module.h siehe struct fwk_module für funtkionen die implementiert werden müssen
--- 

## Aktueller Stand / td:
   - SCP-Firmware grober Aufbau und Funktion verstanden, damit die SCP-Firmware auf meinem Cr-52 laufen kann muss ich 1.Modul ebene passend konfigurieren( als Test hierfür erstmal nur ein Reset-Modul konfiguriert, rest sollte aber schnell gehen, da vorarbeitet hierfür geleistet wurde) und 2. archtitecture spezifischen Code implementieren, konkret wie kriege ich den CR-52 zum laufen/initialisieren. 2 Möglichkeiten dies rauszufinden. Dummy-firmware analysieren oder/(und) im arm manual und doc nachschauen

	-CR-52 Initialisierung:
      ->
	
	-Build Prozess der SCP-Firmware:
      -> Zusammenspiel von CMake und Make
      -> Juno wurde beispielhaft schon gebuilded, das hat geklappt, am ende kriegt man ein .bin .elf .map raus, wie man sie flashed muss man am ende geschaut werden, im Prinzip brauche ich .srec (im Prinzip binary datei mit adressinformationen // ASCII kodiert und mit Prüfsumme)
      -> weiteres kommt sobald arch für armv8-r fertig ist
		
	-Linux:
      ->
		