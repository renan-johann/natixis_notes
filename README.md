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

public class Common {

    /**
     * Waits until the element is visible, then fills it with the given text.
     * @param objectPath the path to the Test Object
     * @param text the text to enter into the element
     * @param timeout (optional) how many seconds to wait for the element to be visible (default = 5)
     */
    @Keyword
    def waitAndFill(String objectPath, String text, int timeout = 5) {
        def testObject = findTestObject(objectPath)
        WebUI.waitForElementVisible(testObject, timeout, FailureHandling.STOP_ON_FAILURE)
        WebUI.setText(testObject, text)
    }

    /**
     * Waits until the element is visible and clickable, then clicks it.
     * Accepts dynamic parameters for the Test Object if needed.
     * @param objectPath the path to the Test Object
     * @param params (optional) map of dynamic parameters to be injected into the Test Object
     * @param timeout (optional) how many seconds to wait for the element to be ready (default = 10)
     */
    @Keyword
    def waitAndClick(String objectPath, Map params = null, int timeout = 10) {
        def testObject = params ? findTestObject(objectPath, params) : findTestObject(objectPath)
        WebUI.waitForElementVisible(testObject, timeout, FailureHandling.STOP_ON_FAILURE)
        WebUI.waitForElementClickable(testObject, timeout, FailureHandling.STOP_ON_FAILURE)
        WebUI.click(testObject)
    }

/**
 * Waits until the element is present in the DOM.
 * @param objectPath path to the Test Object
 * @param timeout (optional) time to wait in seconds (default = 5)
 */
@Keyword
def waitForElementPresent(String objectPath, int timeout = 5) {
    def testObject = findTestObject(objectPath)
    WebUI.waitForElementPresent(testObject, timeout, FailureHandling.STOP_ON_FAILURE)
}

/**
 * Returns the text of a visible element.
 * @param objectPath path to the Test Object
 * @param timeout (optional) time to wait for visibility (default = 5)
 * @return String text of the element
 */
@Keyword
def getElementText(String objectPath, int timeout = 5) {
    def testObject = findTestObject(objectPath)
    WebUI.waitForElementVisible(testObject, timeout, FailureHandling.STOP_ON_FAILURE)
    return WebUI.getText(testObject)
}

/**
 * Verifies that the text of a visible element matches the expected text.
 * @param objectPath path to the Test Object
 * @param expectedText the text you expect to see
 * @param timeout (optional) time to wait for visibility (default = 5)
 */
@Keyword
def verifyElementText(String objectPath, String expectedText, int timeout = 5) {
    def testObject = findTestObject(objectPath)
    WebUI.waitForElementVisible(testObject, timeout, FailureHandling.STOP_ON_FAILURE)
    WebUI.verifyMatch(WebUI.getText(testObject), expectedText, false, FailureHandling.STOP_ON_FAILURE)
}

/**
 * Scrolls to the element and clicks on it.
 * @param objectPath path to the Test Object
 * @param timeout (optional) time to wait (default = 5)
 */
@Keyword
def scrollToAndClick(String objectPath, int timeout = 5) {
    def testObject = findTestObject(objectPath)
    WebUI.waitForElementVisible(testObject, timeout, FailureHandling.STOP_ON_FAILURE)
    WebUI.scrollToElement(testObject, timeout)
    WebUI.click(testObject)
}
}

```

