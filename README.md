### Cria em: Keywords/utils/ObjectReferenceValidator.groovy

```sh
*** IN ***

```


```sh

*** IN ***

```

```sh

Path testCaseRoot = Paths.get(RunConfiguration.getProjectDir(), "Test Cases")
Path reportDir = Paths.get(RunConfiguration.getProjectDir(), "Reports")
Files.createDirectories(reportDir)
Path htmlFile = reportDir.resolve("test-case-progress-report.html")

def moduleStats = [:].withDefault {
    [maintenance: 0, 'new-feature': 0, 'needs-maintenance': 0, total: 0]
}
def totalModules = 0
def totalTestCases = 0
def progressSum = 0

Files.walk(testCaseRoot, 2)
    .filter { Files.isDirectory(it) && !it.equals(testCaseRoot) }
    .each { moduleDir ->
        def moduleName = testCaseRoot.relativize(moduleDir).toString()
        def stats = [maintenance: 0, 'new-feature': 0, 'needs-maintenance': 0, total: 0]
        def includeModule = false

        Files.list(moduleDir)
            .filter { Files.isRegularFile(it) && it.toString().endsWith(".tc") }
            .each { filePath ->
                def tagFound = 'needs-maintenance'
                def skip = false

                try {
                    def content = new String(Files.readAllBytes(filePath), "UTF-8")
                    def tagPattern = Pattern.compile("<tag>(.*?)</tag>", Pattern.CASE_INSENSITIVE)
                    def matcher = tagPattern.matcher(content)
                    def tags = []
                    while (matcher.find()) {
                        tags << matcher.group(1).trim().toLowerCase()
                    }

                    if (tags.contains("skip")) {
                        skip = true
                    } else if (tags.contains("maintenance")) {
                        tagFound = "maintenance"
                    } else if (tags.contains("new-feature")) {
                        tagFound = "new-feature"
                    }

                } catch (Exception e) {
                    println "⚠️ Failed to parse: ${filePath.fileName} (${e.message})"
                }

                if (!skip) {
                    stats[tagFound] += 1
                    stats.total += 1
                    includeModule = true
                }
            }

        if (includeModule && stats.total > 0) {
            moduleStats[moduleName] = stats
            totalModules++
            totalTestCases += stats.total
            progressSum += ((stats.maintenance + stats.'new-feature') / stats.total * 100)
        }
    }

def avgProgress = totalModules > 0 ? (progressSum / totalModules).intValue() : 0

// HTML Template
def html = new StringBuilder()
html << """
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Test Coverage Progress Report</title>
    <style>
        body { font-family: Arial, sans-serif; padding: 20px; background: #fff; color: #333; }
        h1 { font-size: 24px; color: #2c3e50; }
        .summary { display: flex; justify-content: space-between; align-items: center; background: #f8f9fa; padding: 10px; border-radius: 8px; margin-bottom: 20px; }
        .bar-container { background: #e0e0e0; border-radius: 20px; overflow: hidden; height: 24px; margin-top: 8px; margin-bottom: 16px; }
        .bar-fill { height: 100%; text-align: right; color: #fff; font-size: 14px; line-height: 24px; padding-right: 8px; }
        .green { background: #4CAF50; }
        .green-dark { background: #2e7d32; }
        .module { margin-bottom: 25px; }
        .counts { font-size: 14px; margin-top: 5px; }
    </style>
</head>
<body>
    <h1>Test Coverage Progress Report</h1>
    <div class="summary">
        <div></div>
        <div style="text-align: right;">
            <strong>Overview:</strong><br>
            Modules: ${totalModules}<br>
            Total Test Cases: ${totalTestCases}<br>
            Average Progress: ${avgProgress}%
        </div>
    </div>
"""

moduleStats.each { module, stats ->
    def reviewed = stats.maintenance + stats.'new-feature'
    def pending = stats.'needs-maintenance'
    def progress = stats.total > 0 ? (reviewed / stats.total * 100).intValue() : 0
    def barColor = (progress == 100) ? "green-dark" : "green"
    def progressLabel = (progress == 100) ? "✅ Concluído" : "${progress}%"

    html << """
    <div class="module">
        <h2>📁 ${module}</h2>
        <div class="counts">
            ✅ Maintenance: ${stats.maintenance} |
            ✨ New Feature: ${stats.'new-feature'} |
            🛠️ Needs Maintenance: ${pending} |
            📊 Total: ${stats.total}
        </div>
        <div class="bar-container">
            <div class="bar-fill ${barColor}" style="width: ${progress}%;">${progressLabel}</div>
        </div>
    </div>
"""
}

html << """
</body>
</html>
"""

Files.write(htmlFile, html.toString().getBytes("UTF-8"))
println "\\n✅ HTML report saved at: ${htmlFile.toUri()}\\n"

```

