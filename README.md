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
        def shouldSkip = false

        try {
            def content = new String(Files.readAllBytes(filePath), "UTF-8")
            def tagPattern = Pattern.compile("<tag>(.*?)</tag>", Pattern.CASE_INSENSITIVE)
            def matcher = tagPattern.matcher(content)
            def tags = []
            while (matcher.find()) {
                tags << matcher.group(1).trim().toLowerCase()
            }

            if (tags.contains('script')) {
                shouldSkip = true
            } else if (tags.contains('maintenance')) {
                tagFound = 'maintenance'
            } else if (tags.contains('new-feature')) {
                tagFound = 'new-feature'
            }

        } catch (Exception e) {
            println "⚠️ Failed to parse: ${filePath.fileName} (${e.message})"
        }

        if (shouldSkip) {
            println "⏭️ Skipping ${filePath.fileName} (tagged as 'script')"
            return
        }

        def relativePath = testCaseRoot.relativize(filePath)
        def parts = relativePath.toString().split(Pattern.quote(File.separator))
        def topFolder = parts.length > 1 ? parts[0] : "Root"

        moduleStats[topFolder][tagFound] += 1
        moduleStats[topFolder]['total'] += 1
    }

println "\n📈 Final Test Case Tag Summary:\n"

def html = new StringBuilder()
html << "<html><head><style>"
html << "body { font-family: sans-serif; background: #f4f4f4; padding: 20px; }"
html << ".module { background: #fff; margin-bottom: 16px; padding: 12px; border-radius: 8px; box-shadow: 0 0 5px #ccc; }"
html << ".bar { height: 20px; background: #e0e0e0; border-radius: 4px; overflow: hidden; }"
html << ".fill { height: 100%; background: #4caf50; text-align: right; padding-right: 5px; color: white; font-size: 12px; }"
html << "</style></head><body>"
html << "<h2>📊 Test Case Maintenance Report</h2>"

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

    html << "<div class='module'>"
    html << "<strong>📁 ${module}</strong><br/>"
    html << "✔️ Maintenance: ${stats.maintenance} | ✨ New Feature: ${stats.'new-feature'} | 🛠️ Needs Maintenance: ${pending} | Total: ${stats.total}<br/>"
    html << "<div class='bar'><div class='fill' style='width:${progress}%;'>${progress}%</div></div>"
    html << "</div>"
}

html << "</body></html>"

def reportPath = Paths.get(RunConfiguration.getProjectDir(), "Reports", "test-maintenance-report.html")
Files.createDirectories(reportPath.getParent())
Files.write(reportPath, html.toString().getBytes("UTF-8"))

println "📄 HTML report saved at: ${reportPath.toAbsolutePath()}"

```

