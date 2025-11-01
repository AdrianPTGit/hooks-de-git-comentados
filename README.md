# Sí — los git hooks pueden programarse en cualquier lenguaje

Los **hooks de Git** son simplemente ejecutables que Git llama en ciertos momentos (`pre-commit`, `commit-msg`, `pre-push`, etc.).  
No están restringidos a **bash**: pueden ser scripts en **Python**, **Node.js**, **Ruby**, **Perl**, **PowerShell**, binarios compilados, o incluso `.bat`/`.cmd` en Windows, siempre que sean ejecutables y devuelvan el código de salida apropiado.

## Cómo hacerlo (pasos básicos)

1. Crea el archivo en `.git/hooks/` con el nombre del hook (por ejemplo, `.git/hooks/pre-commit`).
2. Añade la *shebang* adecuada en la primera línea (por ejemplo, `#!/usr/bin/env python3` o `#!/usr/bin/env node`).
3. Hazlo ejecutable:

   ```bash
   chmod +x .git/hooks/pre-commit
   ```

4. El hook debe salir con código `0` para `continuar`; cualquier código `distinto` de `0 aborta la operación de Git`.

### Ejemplo: con Python (`pre-commit`):

```pythonchmod +x .git/hooks/pre-commit
#!/usr/bin/env python3
import sys
import subprocess

# Rechaza commits si hay ficheros con "TODO"
changed = subprocess.run(["git", "diff", "--cached", "--name-only"], capture_output=True, text=True)
for f in changed.stdout.splitlines():
    try:
        with open(f, "r", encoding="utf-8") as fh:
            if "TODO" in fh.read():
                print(f"Encontrado TODO en {f} — cancela commit")
                sys.exit(1)
    except FileNotFoundError:
        pass

sys.exit(0)
```
## Consideraciones para Windowschmod +x .git/hooks/pre-commit

En **Windows** puedes usar archivos `.bat` / `.cmd` o **PowerShell**.  
**Git for Windows** puede ejecutar scripts POSIX si usas **Git Bash**.

También puedes colocar un ejecutable (`.exe`) como hook.

---

## Buenas prácticas

- Evita dependencias que no estén disponibles en el entorno **CI** o en los equipos de otros colaboradores.  
- Para proyectos con varios desarrolladores, usa `core.hooksPath` para apuntar a un directorio versionado con hooks compartidos (por ejemplo):

  ```bash
  git config core.hooksPath .githooks
  ```
- y así no depender de `.git/hooks` local.

- Para proyectos JavaScript, hay herramientas como `husky` que facilitan la gestión y distribución de hooks (se instalan en `node_modules` y se configuran desde `package.json`).

### Ejemplo simple en Node.js (pre-commit):chmod +x .git/hooks/pre-commit

```javascript
Ejemplo simple en Node.js (pre-commit)

#!/usr/bin/env node
const { execSync } = require('child_process');

const files = execSync('git diff --cached --name-only').toString().trim().split('\n');
for (const f of files) {
  if (!f) continue;
  const content = require('fs').readFileSync(f, 'utf8');
  if (content.includes('console.log')) {
    console.error(`Encontrado console.log en ${f} — cancela commit`);
    process.exit(1);
  }
}
process.exit(0);

```
### Ejemplo: pre-commit hook en Java

> Queremos crear un hook que impida el commit si algún archivo contiene la palabra TODO.

## 1. Código Java del hook

- Guarda este código como `PreCommitHook.java` (en cualquier carpeta, por ejemplo en el propio `.git/hooks`):

```java
import java.io.*;
import java.util.*;

public class PreCommitHook {
    public static void main(String[] args) {
        try {
            // Ejecuta 'git diff --cached --name-only' para obtener los archivos en el commit
            Process process = new ProcessBuilder("git", "diff", "--cached", "--name-only")
                    .redirectErrorStream(true)
                    .start();

            BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
            List<String> files = new ArrayList<>();
            String line;
            while ((line = reader.readLine()) != null) {
                if (!line.isEmpty()) files.add(line);
            }
            process.waitFor();

            boolean error = false;

            for (String file : files) {
                File f = new File(file);chmod +x .git/hooks/pre-commit
                if (!f.exists()) continue;

                try (BufferedReader fr = new BufferedReader(new FileReader(f))) {
                    String content;
                    while ((content = fr.readLine()) != null) {
                        if (content.contains("TODO")) {
                            System.err.println("Encontrado TODO en: " + file);
                            error = true;
                            break;
                        }import java.io.*;
import java.util.*;

public class PreCommitHook {
    public static void main(String[] args) {
        try {
            // Ejecuta 'git diff --cached --name-only' para obtener los archivos en el commit
            Process process = new ProcessBuilder("git", "diff", "--cached", "--name-only")
                    .redirectErrorStream(true)
                    .start();

            BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
            List<String> files = new ArrayList<>();
            String line;
            while ((line = reader.readLine()) != null) {
                if (!line.isEmpty()) files.add(line);
            }
            process.waitFor();

            boolean error = false;

            for (String file : files) {
                File f = new File(file);
                if (!f.exists()) continue;

                try (BufferedReader fr = new BufferedReader(new FileReader(f))) {
                    String content;
                    while ((content = fr.readLine()) != null) {
                        if (content.contains("TODO")) {
                            System.err.println("Encontrado TODO en: " + file);
                            error = true;
                            break;
                        }
                    }
                }
            }
chmod +x .git/hooks/pre-commit
            if (error) {
                System.err.println("Commit cancelado. Elimina los TODO antes de confirmar.");
                System.exit(1);
            } else {
                System.out.println("Todo correcto. Commit permitido.");
                System.exit(0);
            }

        } catch (Exception e) {
            System.err.println("Error ejecutando hook: " + e.getMessage());
            System.exit(1);
        }
    }
}
                    }
                }
            }

            if (error) {
                System.err.println("Commit cancelado. Elimina los TODO antes de confirmar.");
                System.exit(1);
            } else {
                System.out.println("Todo correcto. Commit permitido.");
                System.exit(0);
            }chmod +x .git/hooks/pre-commit

        } catch (Exception e) {
            System.err.println("Error ejecutando hook: " + e.getMessage());
            System.exit(1);
        }
    }
}
```
## 2. Compílalo

Desde la carpeta donde guardaste el archivo:
```bash
javac PreCommitHook.java
```
- Esto generará `PreCommitHook.class`.
- 
## 3. Crea el script del hook en `.git/hooks/pre-commit`

- Ahora, crea el archivo `.git/hooks/pre-commit` con este contenido:

  ```bash
# Ejecuta el hook Java

```bash
#!/usr/bin/env bash
# Ejecuta el hook Java
java -cp "$(dirname "$0")" PreCommitHook
```
- Hazlo ejecutable:

```bash
chmod +x .git/hooks/pre-commit
```
## 4. Prueba

### 1. Añade un archivo con la palabra “TODO”.

### 2. Haz un commit:

```bash
git add .
git commit -m "test"
```

### 3. Verás algo como:

```bash
Encontrado TODO en: src/Main.java
Commit cancelado. Elimina los TODO antes de confirmar.
```
