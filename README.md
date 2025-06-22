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

    static String currentTestCaseName = ""
    static long testStartTime = 0
    static long suiteStartTime = 0

    // Armazena os resultados detalhados
    static List<Map> testResults = []

    @BeforeTestSuite
    def beforeTestSuite(TestSuiteContext testSuiteContext) {
        suiteStartTime = System.currentTimeMillis()
    }

    @BeforeTestCase
    def beforeTestCase(TestCaseContext testCaseContext) {
        currentTestCaseName = testCaseContext.getTestCaseId().tokenize('/').last()
        testStartTime = System.currentTimeMillis()

        KeywordUtil.logInfo("🚀 Starting Test Case: " + currentTestCaseName)

        WebUI.openBrowser('')
        WebUI.navigateToUrl(GlobalVariable.Link_EN)
        WebUI.waitForPageLoad(GlobalVariable.timeout)
        WebUI.maximizeWindow()
    }

    @AfterTestCase
    def afterTestCase(TestCaseContext testCaseContext) {
        long testEndTime = System.currentTimeMillis()
        long duration = (testEndTime - testStartTime) / 1000

        String status = testCaseContext.getTestCaseStatus()
        def timestamp = new Date().format("yyyyMMdd_HHmmss")

        def reportPath = RunConfiguration.getProjectDir() + "/Reports/_Screenshots/"
        new File(reportPath).mkdirs()
        def screenshotPath = "${reportPath}${currentTestCaseName}_${status}_${timestamp}.png"
        WebUI.takeScreenshot(screenshotPath)

        KeywordUtil.logInfo("📸 Screenshot saved: " + screenshotPath)

        // Adiciona o resultado do teste na lista
        testResults.add([
            name: currentTestCaseName,
            status: status,
            duration: duration
        ])

        WebUI.closeBrowser()
    }

    @AfterTestSuite
    def afterTestSuite(TestSuiteContext testSuiteContext) {
        long totalDuration = (System.currentTimeMillis() - suiteStartTime) / 1000

        KeywordUtil.logInfo("📋 Test Suite Summary:")

        testResults.each { result ->
            def emoji = result.status == 'PASSED' ? "✅" : "❌"
            KeywordUtil.logInfo("${emoji} ${result.name} — ${result.status} (${result.duration}s)")
        }

        KeywordUtil.logInfo("⏱️ Total Suite Duration: ${totalDuration}s")
    }
}
```

