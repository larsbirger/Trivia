# 🚀 Java Version Switching Guide (Windows PowerShell)

Use this procedure when Maven or your IDE reports an `invalid target release` or when you need to ensure you are using **JDK 17**.

## 1. Identify Current Java Locations

First, check which versions of Java are currently installed and where they live:

```powershell
where.exe java
```

*This will list all paths. Look for the one containing `jdk-17`.*

## 2. Set Java 17 for the Current Session

Copy and paste these two lines into your PowerShell terminal. 
> **Note:** Update the path in the first line if your JDK 17 is installed in a different folder (e.g., `Adoptium` or `Zulu`).

```powershell
# Set the home directory for Java 17
$env:JAVA_HOME = "C:\Program Files\Java\jdk-17"

# Move Java 17 to the front of your Path
$env:Path = "$env:JAVA_HOME\bin;$env:Path"
```

## 3. Verify the Switch

Confirm that the system is now "pointing" to the correct version:

```powershell
java -version
mvn -version
```

*The output should now explicitly state **Java version: 17**.*

---

## ⚠️ Important Notes

* **Temporary vs. Permanent:** This method is **temporary**. It only stays active for the specific PowerShell window you are currently using. If you close the window, you must run the commands again.
* **Maven Sync:** After running these commands, you can immediately run `mvn test` or `mvn clean install` in the same window.
* **IDE Sync:** If your IDE (VS Code/IntelliJ) is still showing errors, you may need to restart the IDE or update the **Project SDK** settings in the GUI, as IDEs sometimes ignore the terminal's environment variables.

---

## Pro-Tip: Create a "Switch" Script

If you do this often, save those two lines into a file named `setjava17.ps1`. Then, whenever you start working, you can just run:

```powershell
./setjava17.ps1
```
