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

@AfterTestCase
def afterTestCase(TestCaseContext testCaseContext) {
    def status = testCaseContext.getTestCaseStatus()
    def timestamp = new Date().format("yyyyMMdd_HHmmss")
    def reportPath = RunConfiguration.getProjectDir() + "/Reports/_Screenshots/"
    new File(reportPath).mkdirs()

    def screenshotPath = "${reportPath}${currentTestCaseName}_${status}_${timestamp}.png"
    WebUI.takeScreenshot(screenshotPath)

    long durationMs = testCaseContext.getEndTime() - testCaseContext.getStartTime()
    long seconds = (durationMs / 1000) % 60
    long minutes = (durationMs / 1000) / 60

    def formattedDuration = minutes > 0 ? "${minutes} min ${seconds} s" : "${seconds} s"

    KeywordUtil.logInfo("✅ ${currentTestCaseName} — ${status} (${formattedDuration})")
    KeywordUtil.logInfo("📸 Screenshot saved: " + screenshotPath)

    WebUI.closeBrowser()
}

```

