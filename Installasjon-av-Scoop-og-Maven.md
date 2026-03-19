# 🍦 Installasjon av Scoop og Maven

Scoop er en pakkebehandler for Windows som er utviklet for å være minimalistisk og enkel å bruke fra kommandolinjen.

### 1. Åpne PowerShell

Du trenger ikke nødvendigvis kjøre som administrator for Scoop (det er faktisk anbefalt å kjøre som vanlig bruker), men du må tillate at skripter kjøres:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### 2. Installer Scoop

Kopier og lim inn denne kommandoen for å laste ned og installere Scoop:

```powershell
irm get.scoop.sh | iex
```

* **Hva skjer nå?** Scoop installeres i din brukermappe (`C:\Users\Navn\scoop`), noe som betyr at den ikke forsøpler systemmappene dine.

---

### 3. Legg til "Main" bucket (Valgfritt)

Scoop bruker "buckets" (lagre) for å finne programmer. Hovedlagret er vanligvis installert som standard, men du kan sikre deg ved å kjøre:

```powershell
scoop bucket add main
```

### 4. Installer Apache Maven

Nå kan du installere Maven med en enkel kommando:

```powershell
scoop install maven
```

* **Bonus:** Scoop vil merke om du mangler Java (JDK) og kan ofte foreslå å installere det for deg (f.eks. `scoop install openjdk17`).

---

### 5. Bekreft installasjonen

Siden Scoop oppdaterer `PATH` for deg umiddelbart i den aktive økten, kan du sjekke versjonen med en gang:

```powershell
mvn -version
```

#### feilsøke

##### scoop : The term 'scoop' is not recognized as the name of a cmdlet

Dersom 'scoop' ikke er gjenkjent som en cmdlet, så kan dette være fordi miljøvariablene er utdaterte.
du kan tvinge de til å oppdatere med å kjøre:

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","User") + ";" + [System.Environment]::GetEnvironmentVariable("Path","Machine")
```

Dette tvinger en oppdatering i miljøvariablene.

Hvis kommandoen fortsatt ikke finnes, må vi sjekke om Scoop ble installert i standardmappen. Kjør denne kommandoen:

```powershell
test-path "$env:USERPROFILE\scoop\shims\scoop.ps1"
```

Hvis denne returnerer True, er Scoop installert, men stien mangler i PATH.

Hvis den returnerer False, feilet installasjonen (kanskje pga. brannmur eller rettigheter). Da må du prøve å installere på ny.

### Hvorfor bruke Scoop fremfor Chocolatey?

| Funksjon | Chocolatey | Scoop |
| :--- | :--- | :--- |
| **Rettigheter** | Krever ofte Admin | Kjører som vanlig bruker |
| **Installasjonssted** | `C:\ProgramData` | `C:\Users\Brukernavn\scoop` |
| **Ryddighet** | Kan spre filer i systemet | Holder alt i én mappe |

---

### 💡 Tips for senere

For å oppdatere Maven (eller andre programmer du har installert med Scoop) senere, skriver du bare:

```powershell
scoop update maven
```
