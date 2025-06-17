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
@Keyword
static void verifyLoginLanguage(String expectedLanguage) {
    String title = WebUI.getText(LoginPage.getLoginTitle()).trim()
    String loginLabel = WebUI.getText(LoginPage.getLoginLabel()).trim()
    String passwordLabel = WebUI.getText(LoginPage.getPasswordLabel()).trim()

    switch (expectedLanguage.toLowerCase()) {
        case 'en':
            assert title == 'INTERNAL USER IDENTIFICATION'
            assert loginLabel == 'Login'
            assert passwordLabel == 'Password'
            break

        case 'fr':
            assert title == 'IDENTIFICATION UTILISATEUR INTERNE'
            assert loginLabel == 'Identifiant utilisateur interne'
            assert passwordLabel == 'Mot de passe'
            break

        case 'pt':
            assert title == 'IDENTIFICAÇÃO'
            assert loginLabel == 'Identificador'
            assert passwordLabel == 'Senha'
            break

        default:
            KeywordUtil.markFailed("Unsupported language: $expectedLanguage")
    }
}
```

```sh
LoginActions.verifyLoginLanguage('en')
WebUI.selectOptionByValue(findTestObject(LoginConstants.LANGUAGE_DROPDOWN), 'fr', false)
WebUI.waitForPageLoad(10)
LoginActions.verifyLoginLanguage('fr')
WebUI.selectOptionByValue(findTestObject(LoginConstants.LANGUAGE_DROPDOWN), 'pt', false)
WebUI.waitForPageLoad(10)
LoginActions.verifyLoginLanguage('pt')

static TestObject getLoginTitle() {
    return findTestObject(LoginConstants.LOGIN_TITLE)
}
```

validate_login_language_switch

```sh
    static TestObject getLoginTitle() {
        return findTestObject(LoginConstants.LOGIN_TITLE)
    }

    static TestObject getLoginLabel() {
        return findTestObject(LoginConstants.LOGIN_LABEL)
    }

    static TestObject getPasswordLabel() {
        return findTestObject(LoginConstants.PASSWORD_LABEL)
    }
```
```sh
    // Labels for language validation
    static final String LOGIN_TITLE       = 'login_page/title_login'
    static final String LOGIN_LABEL       = 'login_page/label_login'
    static final String PASSWORD_LABEL    = 'login_page/label_password'
    static final String LANGUAGE_DROPDOWN = 'login_page/select_language'

```

```sh
pipeline {
    agent any

    stages {
        stage('Checkout P2P project') {
            steps {
                cleanWs()
                git credentialsId: '40130d40-5b6c-4c9d-9314-ae82746b7456',
                    url: 'https://bitbucket-mut.mycloud.intrabpce.fr/scm/gi5/p2p.git',
                    branch: 'dev'  // <- Altera aqui para o nome real do teu branch
            }
        }

        stage('Run Katalon Login Suite') {
            steps {
                bat '"C:\\Automation\\Runtime\\Katalon_Studio_Engine_Windows_64-8.1.0\\katalonc.exe" -noSplash -runMode=console -projectPath="C:\\path\\to\\your\\project\\p2p.prj" -retry=0 -testSuitePath="Test Suites/login/validate_login_language_switch" -executionProfile="default" -browserType="Chrome" --config -reportFolder="Reports" -reportFileName="report"'
            }
        }
    }

    post {
        always {
            junit '**/Reports/**/*.xml'
        }
    }
}

```
