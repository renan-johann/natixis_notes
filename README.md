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

String userName = testData.getValue(1, rowIndex)
String role     = testData.getValue(4, rowIndex)  // 4ª coluna = Role

// ...

if (messageImport) {
    KeywordUtil.markFailed("❌ User \"${userName}\" (${role}) SHOULD NOT have access to Import Catalogs")
    WebUI.takeScreenshot(evidencePath + "ERROR_${userName}_${role}_CanImportCatalogs.png")
} else {
    KeywordUtil.markPassed("✅ User \"${userName}\" (${role}) correctly blocked from Import Catalogs")
    WebUI.takeScreenshot(evidencePath + "Unauthorized_${userName}_${role}_CantImportCatalogs.png")
}

```

