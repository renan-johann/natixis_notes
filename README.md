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


```sh



```



```sh
// Caminho da raiz do projeto
Path testCaseRoot = Paths.get(RunConfiguration.getProjectDir(), "Test Cases")

// Palavras-chave válidas que representam status
def maintenanceToken = "AutomationStatus.MAINTENANCE"
def newFeatureToken = "AutomationStatus.NEW_FEATURE"

// Estatísticas agrupadas por pasta principal
def moduleStats = [:].withDefault { 
    [maintenance: 0, 'new-feature': 0, 'needs-maintenance': 0, total: 0] 
}

// Percorre todos os arquivos de teste (.groovy)
Files.walk(testCaseRoot)
    .filter { Files.isRegularFile(it) && it.toString().endsWith(".groovy") }
    .each { Path filePath ->
        def lines = Files.readAllLines(filePath)
        def status = 'needs-maintenance' // padrão, se nenhum status for detectado

        lines.each { line ->
            def trimmed = line.trim()
            if (trimmed == maintenanceToken) {
                status = 'maintenance'
            } else if (trimmed == newFeatureToken) {
                status = 'new-feature'
            }
        }

        // Detecta a pasta principal (módulo)
        def relativePath = testCaseRoot.relativize(filePath)
        def parts = relativePath.toString().split(Pattern.quote(File.separator))
        def topFolder = parts.length > 1 ? parts[0] : "Root"

        // Atualiza os contadores
        moduleStats[topFolder][status] += 1
        moduleStats[topFolder]['total'] += 1
    }

// Exibe o relatório final no console
println "\n📊 Test Case Maintenance Summary (based on AutomationStatus constants)\n"

moduleStats.each { module, stats ->
    def reviewed = stats.maintenance + stats.'new-feature'
    def pending = stats.'needs-maintenance'
    def progress = stats.total > 0 ? (reviewed / stats.total * 100).round(2) : 0

    println "📁 Module: ${module}"
    println "  • Total Test Cases: ${stats.total}"
    println "  • Maintenance: ${stats.maintenance}"
    println "  • New Feature: ${stats.'new-feature'}"
    println "  • Needs Maintenance: ${pending}"
    println "  • Progress: ${progress}%\n"
}

```


```sh

Path testCaseRoot = Paths.get(RunConfiguration.getProjectDir(), "Test Cases")

// Estrutura de contagem por pasta principal
def moduleStats = [:].withDefault { 
    [maintenance: 0, 'new-feature': 0, 'needs-maintenance': 0, total: 0] 
}

// Percorre todos os arquivos .tc dentro da pasta "Test Cases"
Files.walk(testCaseRoot)
    .filter { Files.isRegularFile(it) && it.toString().endsWith(".tc") }
    .each { Path filePath ->
        def status = 'needs-maintenance' // default
        try {
            def dbFactory = DocumentBuilderFactory.newInstance()
            def dBuilder = dbFactory.newDocumentBuilder()
            def doc = dBuilder.parse(filePath.toFile())
            doc.documentElement.normalize()
            def tagNodes = doc.getElementsByTagName("Tag")

            if (tagNodes.getLength() > 0) {
                for (int i = 0; i < tagNodes.getLength(); i++) {
                    def tagValue = tagNodes.item(i).getTextContent().trim()
                    if (tagValue == "maintenance") {
                        status = "maintenance"
                        break
                    } else if (tagValue == "new-feature") {
                        status = "new-feature"
                        break
                    }
                }
            }
        } catch (Exception e) {
            status = "needs-maintenance"
        }

        // Detecta o módulo (pasta principal)
        def relativePath = testCaseRoot.relativize(filePath)
        def parts = relativePath.toString().split(Pattern.quote(File.separator))
        def topFolder = parts.length > 1 ? parts[0] : "Root"

        // Atualiza as estatísticas
        moduleStats[topFolder][status] += 1
        moduleStats[topFolder]['total'] += 1
    }

// Exibe o relatório no console
println "\n📊 Test Case Maintenance Summary (based on .tc tags)\n"

moduleStats.each { module, stats ->
    def reviewed = stats.maintenance + stats.'new-feature'
    def pending = stats.'needs-maintenance'
    def progress = stats.total > 0 ? (reviewed / stats.total * 100).round(2) : 0

    println "📁 Module: ${module}"
    println "  • Total Test Cases: ${stats.total}"
    println "  • Maintenance: ${stats.maintenance}"
    println "  • New Feature: ${stats.'new-feature'}"
    println "  • Needs Maintenance: ${pending}"
    println "  • Progress: ${progress}%\n"
}

```

