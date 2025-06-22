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
class TestListener {

    static String currentTestName = ""

    @BeforeTestCase
    def beforeTestCase(TestCaseContext testCaseContext) {
        WebUI.openBrowser('')
        WebUI.navigateToUrl(GlobalVariable.Link_EN)
        WebUI.waitForPageLoad(GlobalVariable.WaitForLoadTimeout)
        WebUI.maximizeWindow()

        currentTestName = testCaseContext.getTestCaseId().tokenize('/').last()
        KeywordUtil.logInfo("🟢 Starting Test Case: " + currentTestName)
    }

    @AfterTestCase
    def afterTestCase(TestCaseContext testCaseContext) {
        def status = testCaseContext.getTestCaseStatus()
        def timestamp = new Date().format("yyyyMMdd_HHmmss")

        def reportPath = RunConfiguration.getProjectDir() + "/Reports/Screenshots/"
        new File(reportPath).mkdirs()

        def screenshotPath = "${reportPath}${currentTestName}_${status}_${timestamp}.png"
        WebUI.takeScreenshot(screenshotPath)

        KeywordUtil.logInfo("📸 Screenshot saved: " + screenshotPath)

        WebUI.closeBrowser()
    }

    @BeforeTestSuite
    def beforeTestSuite(TestSuiteContext testSuiteContext) {
        KeywordUtil.logInfo("🚀 Starting Test Suite: " + testSuiteContext.getTestSuiteId())
    }

    @AfterTestSuite
    def afterTestSuite(TestSuiteContext testSuiteContext) {
        KeywordUtil.logInfo("🏁 Finished Test Suite: " + testSuiteContext.getTestSuiteId())
    }
}
```

