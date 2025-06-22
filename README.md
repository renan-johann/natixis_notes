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

    /**
     * Variável para guardar o nome do teste em execução
     */
    static String currentTestCaseName = ""

    /**
     * Executa antes da suíte de testes iniciar
     */
    @BeforeTestSuite
    def beforeTestSuite() {
        WebUI.openBrowser('') // Abre navegador vazio
        WebUI.navigateToUrl('https://harmoni.p2p.dev.nporeto.com?lang=en') // Vai pra URL com idioma inglês
        WebUI.waitForPageLoad(GlobalVariable.defaultTimeout) // Espera página carregar com timeout global
        WebUI.maximizeWindow() // Maximiza janela
    }

    /**
     * Executa depois que a suíte de testes terminar
     */
    @AfterTestSuite
    def afterTestSuite() {
        WebUI.closeBrowser() // Fecha o navegador no final da suíte
    }

    /**
     * Captura o nome do teste antes de cada Test Case rodar
     */
    @BeforeTestCase
    def beforeTestCase(TestCaseContext testCaseContext) {
        currentTestCaseName = testCaseContext.getTestCaseId().split('/').last()
        KeywordUtil.logInfo("🔍 Starting Test Case: " + currentTestCaseName)
    }

    /**
     * Executa após cada Test Case. Tira screenshot e salva em pasta customizada.
     */
    @AfterTestCase
    def afterTestCase(TestCaseContext testCaseContext) {
        def status = testCaseContext.getTestCaseStatus() // PASSED ou FAILED
        def timestamp = new Date().format("yyyyMMdd_HHmmss")

        def reportPath = RunConfiguration.getProjectDir() + "/Reports/_Screenshots/"
        new File(reportPath).mkdirs() // Cria pasta se não existir

        def screenshotPath = reportPath + "${currentTestCaseName}_${status}_${timestamp}.png"
        WebUI.takeScreenshot(screenshotPath)

        KeywordUtil.logInfo("📸 Screenshot saved: " + screenshotPath)
    }
}
```

