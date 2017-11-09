# Typesafe Builder

Reprenons là où nous nous étions arrêtés lors de la partie 1

```kotlin runnable
import java.io.File

class FileBuilder(private val rootDir: File) {
    init {
        rootDir.mkdirs()
    }

    fun directory(name: String) {
        File(rootDir, name).mkdirs()
    }
}

fun main(args: Array<String>) {
    println("Hello Kotlin")
    val rootDir = File("test")

    val builder = FileBuilder(rootDir)
    builder.directory("a")
    builder.directory("b")
    builder.directory("c")

    // oups !! nous n'avons plus d'instance 'a'
    // val a1 = File(a, "a1")
    // a1.mkdirs()
    // val a2 = File(a, "a2")
    // a2.mkdirs()

    val process = Runtime.getRuntime().exec("ls -R ${rootDir.canonicalPath}")
    println(process.inputStream.reader().readLines().joinToString("\n"))
    process.waitFor()
}
```

Pour construire `File(a, "a1")` nous avons besoin d'un second `FileBuilder`

[TODO] Modifiez la méthode `fun directory(name: String)` pour retourner un nouveau `FileBuilder`. 
Utilisez ce `FileBuilder` pour construire "a1" et "a2"

Vous devriez maintenant avoir quelque chose de similaire à : 

```kotlin runnable
import java.io.File

class FileBuilder(private val rootDir: File) {
    init {
        rootDir.mkdirs()
    }

    fun directory(name: String): FileBuilder {
        val file = File(rootDir, name)
        return FileBuilder(file)
    }
}

fun main(args: Array<String>) {
    println("Hello Kotlin")
    val rootDir = File("test")

    val builder = FileBuilder(rootDir)
    val a = builder.directory("a")
    builder.directory("b")
    builder.directory("c")

    a.directory("a1")
    a.directory("a2")

    val process = Runtime.getRuntime().exec("ls -R ${rootDir.canonicalPath}")
    println(process.inputStream.reader().readLines().joinToString("\n"))
    process.waitFor()
}
```

Pour la prochaine étape nous allons "inverser" les choses. C'est à dire qu'au lieu de retourner un `FileBuilder`
la méthode `fun directory(name: String)` va prendre en paramètre un bloc de code de type `FileBuilder.() -> Unit`.
De plus, puisque c'est ce bloc de code qui va effectuer les traitements, nous n'avons plus besoin de retourner un `FileBuilder`

[TODO] modifiez la méthode `fun directory(name: String)` pour ajouter (et utiliser) le paramètre `FileBuilder.() -> Unit`.
Prenez exemple sur la page précédente si besoin.

[TODO] modifiez aussi la méthode `main` en conséquence.

Voici un exemple de solution : 
```kotlin runnable
import java.io.File

class FileBuilder(private val rootDir: File) {
    init {
        rootDir.mkdirs()
    }

    fun directory(name: String, block: FileBuilder.() -> Unit) {
        val file = File(rootDir, name)
        FileBuilder(file).block()
    }
}

fun main(args: Array<String>) {
    println("Hello Kotlin")
    val rootDir = File("test")

    val builder = FileBuilder(rootDir)
    builder.directory("a", {
        directory("a1", {})
        directory("a2", {})
    })
    builder.directory("b", {})
    builder.directory("c", {})

    val process = Runtime.getRuntime().exec("ls -R ${rootDir.canonicalPath}")
    println(process.inputStream.reader().readLines().joinToString("\n"))
    process.waitFor()
}
```

Bon, cela commence à ressembler à notre exemple en première page mais il y a encore quelques ajustements à faire.

Puisque la méthode `directory` attend désormais deux paramètres, nous sommes obligés d'écrire `directory("a1", {})`.
Nous allons changer cela très simplement en ajoutant une valeur par défaut.

[TODO] ajoutez une valeur par défaut sur le paramètre `block`

Voici un exemple de solution : 
```kotlin runnable
import java.io.File

class FileBuilder(private val rootDir: File) {
    init {
        rootDir.mkdirs()
    }

    fun directory(name: String, block: FileBuilder.() -> Unit = {}) {
        val file = File(rootDir, name)
        FileBuilder(file).block()
    }
}

fun main(args: Array<String>) {
    println("Hello Kotlin")
    val rootDir = File("test")

    val builder = FileBuilder(rootDir)
    builder.directory("a", {
        directory("a1")
        directory("a2")
    })
    builder.directory("b")
    builder.directory("c")

    val process = Runtime.getRuntime().exec("ls -R ${rootDir.canonicalPath}")
    println(process.inputStream.reader().readLines().joinToString("\n"))
    process.waitFor()
}
```

Maintenant nous allons nous occuper de la variable `builder`. Pour la faire disparaitre,
nous allons avoir besoin d'un "point d'entrée".
Ce point d'entrée sera une fonction quasiment identique à notre fonction `directory` mais
avec quelques différences : 
- elle sera placée directement dans le fichier (pas dans une classe)
- le `File` qui sera créé n'aura pas de parent donc on utilisera le constructeur `File(String)`
- cette méthode retournera l'objet `File` créé (nous en avons besoin dans le `main`)

[TODO] créez une autre méthode `directory` comme définit ci-dessus.

Voici un exemple de solution : 
```kotlin
fun directory(name: String, block: FileBuilder.() -> Unit = {}): File {
    val file = File(name)
    FileBuilder(file).block()
    return file
}
```

[TODO] maintenant, remplacez le `FileBuilder` de la méthode `main` par ce point d'entrée.
Votre méthode `main` ne doit plus contenir de `FileBuilder`, uniquement des appels à la fonction `directory`

Voici un exemple de solution : 
```kotlin runnable
import java.io.File

class FileBuilder(private val rootDir: File) {
    init {
        rootDir.mkdirs()
    }

    fun directory(name: String, block: FileBuilder.() -> Unit = {}) {
        val file = File(rootDir, name)
        file.mkdirs()
        FileBuilder(file).block()
    }
}

fun directory(name: String, block: FileBuilder.() -> Unit = {}): File {
    val file = File(name)
    file.mkdirs()
    FileBuilder(file).block()
    return file
}

fun main(args: Array<String>) {
    println("Hello Kotlin")

    val rootDir = directory("test", {
        directory("a", {
            directory("a1")
            directory("a2")
        })
        directory("b")
        directory("c")
    })

    val process = Runtime.getRuntime().exec("ls -R ${rootDir.canonicalPath}")
    println(process.inputStream.reader().readLines().joinToString("\n"))
    process.waitFor()
}
```

Il ne reste plus que la touche finale, celle qui fera vraiment ressembler votre code à un Typesafe Builder.
Bonne nouvelle, vous n'avez rien à développer, nous n'avons besoin que d'un sucre syntaxique fourni par le
compilateur Kotlin.

En effet, lorsque le dernier paramètre d'une fonction est un 'function type' il est possible de "sortir" ce 
paramètre des parenthèses. 

Ainsi : 
```kotlin
directory("a", {
    directory("a1")
    directory("a2")
})
```

Est exactement équivalent à :
```kotlin
directory("a") {
    directory("a1")
    directory("a2")
}
```

[TODO] modifiez votre méthode `main` pour "sortir" la lambda en dehors des parenthèses (IntelliJ propose
un 'code assist' pour cela)

Voilà, votre Typesafe Builder est terminé :
```kotlin runnable
import java.io.File

class FileBuilder(private val rootDir: File) {
    init {
        rootDir.mkdirs()
    }

    fun directory(name: String, block: FileBuilder.() -> Unit = {}) {
        val file = File(rootDir, name)
        file.mkdirs()
        FileBuilder(file).block()
    }
}

fun directory(name: String, block: FileBuilder.() -> Unit = {}): File {
    val file = File(name)
    file.mkdirs()
    FileBuilder(file).block()
    return file
}

fun main(args: Array<String>) {
    println("Hello Kotlin")

    val rootDir = directory("test") {
        directory("a") {
            directory("a1")
            directory("a2")
        }
        directory("b")
        directory("c")
    }

    val process = Runtime.getRuntime().exec("ls -R ${rootDir.canonicalPath}")
    println(process.inputStream.reader().readLines().joinToString("\n"))
    process.waitFor()
}
```

Si vous voulez ajouter des fonctionnalités à votre Builder, il suffit d'y ajouter des méthodes : 

```kotlin runnable
import java.io.File
import java.io.Writer

class FileBuilder(private val rootDir: File) {
    init {
        rootDir.mkdirs()
    }

    fun directory(name: String, block: FileBuilder.() -> Unit = {}) {
        val file = File(rootDir, name)
        file.mkdirs()
        FileBuilder(file).block()
    }

    fun file(name: String, block: Writer.() -> Unit = {}) {
        val file = File(rootDir, name)
        file.writer().use(block)
    }
}

fun directory(name: String, block: FileBuilder.() -> Unit = {}): File {
    val file = File(name)
    file.mkdirs()
    FileBuilder(file).block()
    return file
}

fun main(args: Array<String>) {
    println("Hello Kotlin")

    val rootDir = directory("test") {
        directory("a") {
            directory("a1")
            directory("a2")
        }
        directory("b")
        directory("c") {
            file("test.txt") {
                write("Hello\n")
                write("World\n")
            }
            file("empty.txt")
        }
    }

    val ls = Runtime.getRuntime().exec("ls -R ${rootDir.canonicalPath}")
    println(ls.inputStream.reader().readLines().joinToString("\n"))
    ls.waitFor()

    println()

    val cat = Runtime.getRuntime().exec("cat ${rootDir.canonicalPath}/c/test.txt")
    println(cat.inputStream.reader().readLines().joinToString("\n"))
    cat.waitFor()
}
```