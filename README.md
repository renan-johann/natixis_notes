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

List<Integer> userIndexes = [5, 2, 10, 3, 12, 16, 4, 1, 17, 6, 14]
String evidencePath = RunConfiguration.getReportFolder() + "/Evidences/"
def testData = findTestData(LoginConstants.CREDENTIALS_DATA_SHEET)

for (int rowIndex : userIndexes) {
    String userName = testData.getValue(1, rowIndex)

    LoginActions.LoginUser(LoginConstants.CREDENTIALS_DATA_SHEET, rowIndex)

    WebUI.navigateToUrl(GlobalVariable.HOMEPAGE_URL)
    UIActions.waitAndClick('menu_harmoni/buttonCatalogs')

    boolean messageImport = UIActions.waitForElementPresent('menu_harmoni/buttonImportCatalogs')

    if (messageImport) {
        KeywordUtil.markFailed("❌ User ${userName} should NOT have access to Import Catalogs")
        WebUI.takeScreenshot(evidencePath + "Error_${userName}_CanImportCatalogs.png")
    } else {
        KeywordUtil.markPassed("✅ User ${userName} correctly blocked from Import Catalogs")
        WebUI.takeScreenshot(evidencePath + "Unauthorized_${userName}_CantImportCatalogs.png")
    }

    WebUI.navigateToUrl(GlobalVariable.LOGIN_URL) // Faz logout
}

```

