### Cria em: Keywords/utils/ObjectReferenceValidator.groovy

```sh
*** IN ***

```


```sh

*** IN ***

```

```sh

        try {
            def content = new String(Files.readAllBytes(filePath), "UTF-8")

            def tagPattern = Pattern.compile("<tag>(.*?)</tag>", Pattern.CASE_INSENSITIVE)
            def matcher = tagPattern.matcher(content)
            def tags = []
            while (matcher.find()) {
                tags << matcher.group(1).trim().toLowerCase()
            }

            if (tags.contains('maintenance')) {
                tagFound = 'maintenance'
            } else if (tags.contains('new-feature')) {
                tagFound = 'new-feature'
            }

        } catch (Exception e) {
            println "⚠️ Failed to parse: ${filePath.fileName} (${e.message})"
        }

```

