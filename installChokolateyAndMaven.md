# 📦 Installasjon av Chocolatey og Maven

Dette dokumentet beskriver hvordan du installerer Chocolatey og Maven via PowerShell.

### 1. Åpne PowerShell som Administrator

For å installere programvare via kommandolinjen, må du ha administrative rettigheter.

* Trykk på **Start-menyen**.
* Søk etter **PowerShell**.
* Høyreklikk og velg **"Kjør som administrator"** (Run as Administrator).

### 2. Installer Chocolatey

Kopier og lim inn følgende kommando i PowerShell-vinduet og trykk **Enter**:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

* **Hva skjer nå?** Skriptet laster ned Chocolatey og setter opp systemet ditt slik at du kan bruke `choco`-kommandoen.

### 3. Verifiser Chocolatey-installasjonen

Lukk PowerShell-vinduet og åpne et **nytt** PowerShell-vindu (som administrator). Skriv:

```powershell
choco -?
```

Hvis du ser en hjelpetekst med versjonsnummer, er Chocolatey klar til bruk.

---

### 4. Installer Apache Maven

Nå som Chocolatey er installert, er det svært enkelt å installere Maven. Kjør følgende kommando:

```powershell
choco install maven -y
```

* `-y` flagget betyr "yes", slik at du slipper å bekrefte installasjonen manuelt.

---

### 5. Oppdater miljøvariabler (Viktig!)

For at Windows skal oppdage den nye `mvn`-kommandoen uten at du trenger å starte maskinen på nytt, kjør denne kommandoen i samme vindu:

```powershell
refreshenv
```

### 6. Bekreft at Maven fungerer

Sjekk at Maven er installert og koblet til riktig Java-versjon:

```powershell
mvn -version
```

Du bør nå se noe lignende som dette:
> Apache Maven 3.x.x ...
> Java version: 17.x.x ...

---

## 🛠 Feilsøking

* **"mvn is not recognized":** Prøv å lukke alle terminalvinduer (og VS Code/IntelliJ) og åpne dem på nytt.
* **Feil Java-versjon:** Hvis Maven bruker feil Java-versjon, kan du bruke din tidligere prosedyre for å sette `$env:JAVA_HOME` til riktig mappe.
