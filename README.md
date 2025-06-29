### Cria em: Keywords/utils/ObjectReferenceValidator.groovy

```sh
*** IN ***

```


```sh

*** IN ***

```

```sh

String baseDir = "Test Cases"
File testCasesDir = new File(RunConfiguration.getProjectDir(), baseDir)

def modules = [:]
def df = new DecimalFormat("0.00")

testCasesDir.eachDir { moduleDir ->
    String moduleName = moduleDir.name
    modules[moduleName] = [total: 0, maintenance: 0, newFeature: 0]

    moduleDir.eachFileRecurse(FileType.FILES) { file ->
        if (file.name.endsWith(".ts")) {
            modules[moduleName].total++

            def tags = []
            def xml = new XmlParser().parse(file)

            xml.depthFirst().findAll { it.name() == 'Tag' }.each { tagNode ->
                def tagText = tagNode.text().trim().toLowerCase()
                println "Found tag: '${tagText}' in file: ${file.name}" // DEBUG
                tags << tagText
            }

            println "Tags for ${file.name}: ${tags}" // DEBUG

            if (tags.contains("maintenance")) {
                modules[moduleName].maintenance++
            } else if (tags.contains("new feature")) {
                modules[moduleName].newFeature++
            }
        }
    }
}

println "\n===== TEST CASE TAG SUMMARY =====\n"

modules.each { module, stats ->
    int total = stats.total
    int maintenance = stats.maintenance
    int newFeature = stats.newFeature
    int needsMaintenance = total - maintenance - newFeature
    double progress = total > 0 ? (maintenance + newFeature) * 100.0 / total : 0.0

    println "📁 Module: ${module}"
    println "• Total Test Cases: ${total}"
    println "• Maintenance: ${maintenance}"
    println "• New Feature: ${newFeature}"
    println "• Needs Maintenance: ${needsMaintenance}"
    println "• Progress: ${df.format(progress)}%"
    println ""
}

```

