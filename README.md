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

Files.walk(testCaseRoot)
    .filter { 
        Files.isRegularFile(it) && it.toString().endsWith(".tc")
    }
    .each { Path filePath ->
        def tagFound = 'needs-maintenance'

        try {
            def content = new String(Files.readAllBytes(filePath), "UTF-8")
            def tagMatches = content.findAll(/<tag>(.*?)<\/tag>/i)  // case-insensitive tag

            def tags = tagMatches.collect { match ->
                match.replaceAll(/<\/?tag>/i, "").trim().toLowerCase()
            }

            if (tags.contains('maintenance')) {
                tagFound = 'maintenance'
            } else if (tags.contains('new-feature')) {
                tagFound = 'new-feature'
            }

        } catch (Exception e) {
            println "⚠️ Failed to parse: ${filePath.fileName} (${e.message})"
        }

        def relativePath = testCaseRoot.relativize(filePath)
        def parts = relativePath.toString().split(Pattern.quote(File.separator))
        def topFolder = parts.length > 1 ? parts[0] : "Root"

        moduleStats[topFolder][tagFound] += 1
        moduleStats[topFolder]['total'] += 1
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

