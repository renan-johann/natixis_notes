### Cria em: Keywords/utils/ObjectReferenceValidator.groovy

```sh
package utils

import com.kms.katalon.core.testobject.ObjectRepository as OR
import com.kms.katalon.core.util.KeywordUtil
import java.nio.file.*
import java.util.regex.*

class ObjectReferenceValidator {

  static void validateAllTestObjectPaths(String baseDirPath = "Scripts") {
    def baseDir = new File(baseDirPath)
    if (!baseDir.exists()) {
      KeywordUtil.markFailed("Diretório ${baseDirPath} não encontrado.")
      return
    }

    def pattern = ~/findTestObject\(["']([^"']+)["']\)/
    def allLines = []

    baseDir.eachFileRecurse(FileType.FILES) { file ->
      if (file.name.endsWith(".groovy")) {
        def lines = file.readLines()
        lines.eachWithIndex { line, index ->
          def matcher = pattern.matcher(line)
          while (matcher.find()) {
            def path = matcher.group(1)
            try {
              OR.findTestObject(path)
            } catch (Exception e) {
              KeywordUtil.markWarning("Objeto NÃO encontrado: ${path} (arquivo: ${file.name}, linha ${index + 1})")
            }
          }
        }
      }
    }

    KeywordUtil.logInfo("Validação concluída.")
  }
}
```

### Como usar:

No console do Katalon, chama:

```sh
utils.ObjectReferenceValidator.validateAllTestObjectPaths()
```



```sh
# ----------------------------------------------------
# Build, cache, and IDE metadata
# ----------------------------------------------------
.gradle/
.build/
bin/
.settings/
.cache/
.classpath
.project
build.gradle
console.properties

# ----------------------------------------------------
# Test execution outputs and runtime files
# ----------------------------------------------------
Reports/
output/
LastRun.xlsx

# ----------------------------------------------------
# Katalon internal temp/test metadata
# ----------------------------------------------------
Libs/
!output/.gitkeep
.settings/internal/com.kms.katalon.composer.testcase.properties
.cache/Keywords/GetLanguage

# ----------------------------------------------------
# System-specific or IDE-specific files (optional)
# ----------------------------------------------------
*.log
*.tmp
.DS_Store
Thumbs.db

# ----------------------------------------------------
# Git metadata (keep .git directory itself)
# ----------------------------------------------------
*.swp
*.swo
```


Title:
Maintenance of Test Cases in BPCE_SI_PR Folder

Description:
Perform maintenance and updates on the following test cases located in the BPCE_SI_PR folder:
	•	approver1_validation
	•	generate_PR_Non_PRTM
	•	generate_PR_non_PRTM_case2
	•	P2P_specialist_validation

The goal is to review and adjust these test cases to ensure proper execution and alignment with the current system behavior. This may include updating test data, fixing broken steps, refactoring logic, or syncing with recent functional changes.
