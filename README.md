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
    static long testCaseStartTime = 0L
    static long suiteStartTime = 0L
    static List<Map> testResults = []

    @BeforeTestSuite
    def beforeTestSuite(TestSuiteContext testSuiteContext) {
        suiteStartTime = System.currentTimeMillis()
        KeywordUtil.logInfo("🟢 Test Suite Started")
    }

    @BeforeTestCase
    def beforeTestCase(TestCaseContext testCaseContext) {
        testCaseStartTime = System.currentTimeMillis()

        // Abre o navegador e navega até a página inicial
        WebUI.openBrowser('')
        WebUI.navigateToUrl(GlobalVariable.Link_EN)
        WebUI.waitForPageLoad(GlobalVariable.defaultTimeout)
        WebUI.maximizeWindow()

        currentTestCaseName = testCaseContext.getTestCaseId().tokenize('/').last()
        KeywordUtil.logInfo("▶️ Starting Test Case: " + currentTestCaseName)
    }

    @AfterTestCase
    def afterTestCase(TestCaseContext testCaseContext) {
        def status = testCaseContext.getTestCaseStatus()
        def timestamp = new Date().format("yyyyMMdd_HHmmss")

        // Caminho para salvar o screenshot
        def reportPath = RunConfiguration.getProjectDir() + "/Reports/_Screenshots/"
        new File(reportPath).mkdirs()
        def screenshotPath = "${reportPath}${currentTestCaseName}_${status}_${timestamp}.png"

        // Captura de tela após o teste
        WebUI.takeScreenshot(screenshotPath)

        KeywordUtil.logInfo("✅ ${currentTestCaseName} - ${status}")
        KeywordUtil.logInfo("📸 Screenshot saved: " + screenshotPath)

        // Adiciona aos resultados
        testResults << [
            name: currentTestCaseName,
            status: status
        ]

        WebUI.closeBrowser()
    }

    @AfterTestSuite
    def afterTestSuite(TestSuiteContext testSuiteContext) {
        long totalDuration = (System.currentTimeMillis() - suiteStartTime) / 1000

        KeywordUtil.logInfo("📊 Test Suite Summary:")
        testResults.each { result ->
            def emoji = result.status == 'PASSED' ? "✅" : "❌"
            KeywordUtil.logInfo("${emoji} ${result.name} - ${result.status}")
        }

        KeywordUtil.logInfo("⏱ Total Suite Duration: ${totalDuration}s")
        KeywordUtil.logInfo("🔚 END Test Suite")
    }
}

```

