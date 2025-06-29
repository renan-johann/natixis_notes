### Cria em: Keywords/utils/ObjectReferenceValidator.groovy

```sh
*** IN ***

```


```sh

*** IN ***

```

```sh

Path testCaseRoot = Paths.get(RunConfiguration.getProjectDir(), "Test Cases")

def moduleStats = [:].withDefault {
    [maintenance: 0, 'new-feature': 0, 'needs-maintenance': 0, total: 0]
}

// Lista apenas os diretórios diretamente dentro de "Test Cases"
Files.newDirectoryStream(testCaseRoot).each { Path moduleDir ->
    if (!Files.isDirectory(moduleDir)) return

    String moduleName = moduleDir.fileName.toString()

    Files.newDirectoryStream(moduleDir).each { Path testCaseFile ->
        if (!Files.isRegularFile(testCaseFile) || !testCaseFile.toString().endsWith(".tc")) return

        def tagFound = 'needs-maintenance'

        try {
            def content = new String(Files.readAllBytes(testCaseFile), "UTF-8")
            def tagMatches = content.findAll(/<tag>(.*?)<\/tag>/i)

            def tags = tagMatches.collect { match ->
                match.replaceAll(/<\/?tag>/i, "").trim().toLowerCase()
            }

            if (tags.contains('maintenance')) {
                tagFound = 'maintenance'
            } else if (tags.contains('new-feature')) {
                tagFound = 'new-feature'
            }

        } catch (Exception e) {
            println "⚠️ Failed to parse: ${testCaseFile.fileName} (${e.message})"
        }

        moduleStats[moduleName][tagFound] += 1
        moduleStats[moduleName]['total'] += 1
    }
}

println "\n📈 Final Test Case Tag Summary:\n"

moduleStats.each { module, stats ->
    def reviewed = stats.maintenance + stats.'new-feature'
    def pending = stats.'needs-maintenance'
    def progress = stats.total > 0 ? ((reviewed / stats.total * 100) as int) : 0
    def remaining = stats.total > 0 ? ((pending / stats.total * 100) as int) : 0

    println "📁 Module: ${module}"
    println "  • Total Test Cases: ${stats.total}"
    println "  • Maintenance: ${stats.maintenance}"
    println "  • New Feature: ${stats.'new-feature'}"
    println "  • Needs Maintenance: ${pending}"
    println "  • Progress: ${progress}%"
    println "  • Remaining: ${remaining}%\n"
}

```

