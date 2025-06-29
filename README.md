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
    .filter { Files.isRegularFile(it) && it.toString().endsWith(".tc") }
    .each { Path filePath ->
        def tagFound = 'needs-maintenance'

        println "🔍 Analyzing file: ${filePath}" // DEBUG 1

        try {
            def content = new String(Files.readAllBytes(filePath), "UTF-8")
            println "📄 Raw content snippet:\n" + content.take(200) + "\n..." // DEBUG 2

            def tagMatches = content.findAll(/<Tag>(.*?)<\/Tag>/)
            println "🏷️ Found raw tag matches: ${tagMatches}" // DEBUG 3

            def tags = tagMatches.collect { match ->
                def inner = match.replaceAll(/<\/?Tag>/, "").toLowerCase().trim()
                return inner
            }

            println "✅ Cleaned tags: ${tags}" // DEBUG 4

            if (tags.contains('maintenance')) {
                tagFound = 'maintenance'
                println "🛠️ Tag classified as: maintenance" // DEBUG 5
            } else if (tags.contains('new-feature')) {
                tagFound = 'new-feature'
                println "🆕 Tag classified as: new-feature" // DEBUG 6
            } else {
                println "⚠️ No recognized tag. Will count as needs-maintenance." // DEBUG 7
            }

        } catch (Exception e) {
            println "❌ Failed to parse: ${filePath.fileName} (${e.message})"
        }

        def relativePath = testCaseRoot.relativize(filePath)
        def parts = relativePath.toString().split(Pattern.quote(File.separator))
        def topFolder = parts.length > 1 ? parts[0] : "Root"

        println "📂 Module (top folder): ${topFolder}" // DEBUG 8

        moduleStats[topFolder][tagFound] += 1
        moduleStats[topFolder]['total'] += 1
        println "📊 Updated stats for ${topFolder}: ${moduleStats[topFolder]}\n" // DEBUG 9
    }

println "\n📈 Final Test Case Tag Summary:\n"

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

