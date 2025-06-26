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

// Importa o arquivo de dados
TestData testData = findTestData('LoginConstants.CREDENTIALS_DATA_SHEET')

// Lista de índices que você quer testar
List<Integer> userIndexes = [2, 4, 5, 8, 10]

for (int rowIndex : userIndexes) {
    // Lê os dados do usuário
    String username = testData.getValue(2, rowIndex)
    String password = testData.getValue(3, rowIndex)
    String role     = testData.getValue(4, rowIndex)

    println "🔄 Testing user at row $rowIndex → Username: $username | Role: $role"

    // Faz login com os dados
    LoginActions.LoginUser(username, password)

    // Ação do teste
    UIActions.waitAndClick('menu_harmoni/buttonCatalogs')

    boolean messageImport = WebUI.verifyElementNotPresent(findTestObject('menu_harmoni/buttonImportCatalogs'), 4)

    if (messageImport == false) {
        KeywordUtil.markFailed("❌ User '$role' should NOT have access to Import Catalogs")
        WebUI.takeScreenshot(RunConfiguration.getReportFolder() + "/Evidences/ErrorUnauthorizedProfilesCanImportCatalogs_${role.replaceAll('\\s+', '_')}.png")
    } else {
        KeywordUtil.markPassed("✅ User '$role' correctly blocked from Import Catalogs")
        WebUI.takeScreenshot(RunConfiguration.getReportFolder() + "/Evidences/UnauthorizedProfilesCantImportCatalogs_${role.replaceAll('\\s+', '_')}.png")
    }

    // Volta para o login (faz o "logout")
    WebUI.navigateToUrl(GlobalVariable.LOGIN_URL)
}

```

