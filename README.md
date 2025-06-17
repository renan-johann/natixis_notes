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
pipeline {
    agent { label 'GIS_SWDCFRNXGI55817' } // mesmo label da VM em uso

    stages {
        stage('Checkout P2P Project') {
            steps {
                cleanWs()
                git credentialsId: '40130d40-5b6c-4c9d-9314-ae82746b7456',
                    url: 'https://bitbucket-mut.mycloud.intrabpce.fr/scm/gi5/p2p.git',
                    branch: 'dev'
            }
        }

        stage('Run Login Suite') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    executeKatalon executeArgs: ''' 
                        -projectPath="D:/jenkins/work/workspace/GI5/P2P Project/P2P" 
                        -browserType="Chrome" 
                        -testSuiteCollectionPath="Test Suites/login/Login TSC" 
                        -noSplash -runMode=console 
                    ''',
                    location: 'C:/Automation/Runtime/Katalon_Studio_Engine_Windows_64-8.1.0',
                    version: '',
                    x11Display: '',
                    xvfbConfiguration: ''
                }
            }
        }
    }

    post {
        always {
            junit '**/Reports/**/JUnit_Report.xml'
        }
    }
}
```
