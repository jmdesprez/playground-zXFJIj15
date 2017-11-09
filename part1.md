# Prérequis
- [IntelliJ](https://www.jetbrains.com/idea/download/index.html#section=linux) 
ou [Eclipse](https://www.eclipse.org/downloads/)
avec le [plugin Kotlin](https://marketplace.eclipse.org/content/kotlin-plugin-eclipse)
- c'est tout :)

# FileBuilder

Le but du TP est de développer un "typesafe builder" permettant de représenter 
une arborescence composée de dossiers, sous-dossiers et fichiers.

```kotlin
fun main(args: Array<String>) {
  fileBuilder(rootFolder) {
    directory("a") {
      file("test.txt") {
        writeText("Hello world!!")
      }
    }
    directory("b") {
      directory("b1") // Empty
      directory("b2") {
        file("templatedemo.txt") {
          generate("rawData.txt", RawType)
        }
        file("dynamicTemplate.txt") {
         val context = mapOf("name" to "super template!!")
         generate("template.txt", MustacheType(context))
        }
      }
    }
  }
}
```

# Hello Kotlin
Commençons par voir quelques spécificités du langage

[TODO] créez un nouveau fichier Kotlin (dans le dossier src)

Contrairement à JAVA un fichier source Kotlin peut contenir autre chose que des classes

[TODO] créez une fonction 'main' en prenant exemple sur le code ci-dessus

[TODO] utilisez la fonction `println` pour afficher un message

[TODO] executez la fonction main. Votre message doit s'afficher dans la console. Si votre IDE ne vous
permet pas d'executer la fonction, vérifier sa signature.

[Mémo] Un fichier Kotlin peut contenir: 
- une ou plusieurs classes
- une ou plusieurs fonctions
- une ou plusieurs variables

[Mémo] le mot clef `static` n'existe pas en Kotlin

# Instances, Variables, Valeurs
## Instances de classes
Pour créer une instance d'une classe, il faut appeler le constructeur de la classe (comme en JAVA) mais en 
omettant le mot clef `new`

[TODO] créez une instance de l'objet `File` en utilisant le constructeur `File(String)` (peu importe
le nom du dossier) affichez son 'canonical path'. Exemple : `println(File(".").canonicalPath)`

[??!!] votre IDE vous proposera très certainement la syntaxe `canonicalPath` au lieu de `getCanonicalPath`
car Kotlin remplace les accès aux getter et setter par le nom de la propriété (mais l'appel au getter/setter
est tout de même effectué !)

## Variables et Valeurs
Kotlin utilise deux mots clefs distincts pour définir des variables : 
- `var login: String = "a149812"` est l'équivalent JAVA de `String login = "a149812";`
- `val login: String = "a149812"` est l'équivalent JAVA de `final String login = "a149812";`

Par ailleurs, le type d'une variable/valeur peut être omis lors de la déclaration s'il peut être 
déterminé avec certitude par le compilateur : 
- `var login = "a149812"` est équivalent à `var login: String = "a149812"`
- `val login = "a149812"` est équivalent à `val login: String = "a149812"`

## Exercice
- créez une instance de `File` appelée `rootDir`
- créez 3 instances de `File` ayant pour parent le dossier `rootDir`. Utilisez le constructeur `File(File, String)`
- dans un des sous-dossiers (peu importe lequel), créez 2 sous-dossiers
- créez physiquement tous ces dossiers en appelant la méthode `mkdirs()`

Voici un exemple de ce que vous devriez avoir : 

``` kotlin runnable
import java.io.File

fun main(args: Array<String>) {
    println("Hello Kotlin")
    val rootDir = File("test")
    rootDir.mkdirs()
    println(rootDir.canonicalPath)
    val a = File(rootDir, "a")
    a.mkdirs()
    val b = File(rootDir, "b")
    b.mkdirs()
    val c = File(rootDir, "c")
    c.mkdirs()
    val a1 = File(a, "a1")
    a1.mkdirs()
    val a2 = File(a, "a2")
    a2.mkdirs()
    
    val process = Runtime.getRuntime().exec("ls -R ${rootDir.canonicalPath}")
    println(process.inputStream.reader().readLines().joinToString("\n"))
    process.waitFor()
}
```
---

# Classes
Nous allons commencer à mettre un peut d'ordre dans notre code en commençant par mettre en place un factory
design pattern. Notre factory sera configurée avec un dossier 'racine' et permettra de créer des `File` qui auront
ce dossier 'racine' comme dossier parent. 

Commencez par créer la classe. Voici un exemple d'implémentation :

```kotlin
class FileBuilder {
    private val rootDir: File
    
    constructor(rootDir: File) {
        this.rootDir = rootDir
        rootDir.mkdirs()
    }
}
```

[Mémo] la visibilité est `public` par défaut

Ormis les spécificités du langage, cette structure devrait vous sembler familière... Une classe avec
des propriétés et un constructeur permettant d'initialiser ces propriétés... En fait, c'est une 
structure tellement courante que Kotlin en propose une version condensée : 

```kotlin
class FileBuilder(private val rootDir: File) {
  init {
    rootDir.mkdirs()
  }
}
``` 

[Note] les deux syntaxes ci-dessus sont exactement équivalente :)

[Note] ce type de constructeur est appelé "[primary constructor](https://kotlinlang.org/docs/reference/classes.html#constructors)"

[Note] `init` vous permet d'executer du code à l'initialisation d'une instance de la classe

Si le corps de la classe est vide (comme pour un 'POJO' par exemple) vous pouvez même omettre les accolades : 

```kotlin
class User(val login: String, val firstname: String, val lastname: String, val age: Int, val isAdmin: Boolean)
```

[??!!] peut-être vous demandez-vous où sont passés les getter/setter :) Ils sont automatiquement générés
par le langage (mais surchargeables si besoin)

[TODO] ajoutez une méthode `directory` dans votre factory pour créer un `File` (ayant pour parent `rootDir`)
et créez ce sous-dossier sur le disque (en appelant la méthode `mkdirs`)

Si vous essayez d'utiliser la factory dans votre méthode `main` vous allez rencontrer un petit problème : 

```kotlin runnable
import java.io.File

class FileBuilder(private val rootDir: File) {
    init {
        this.rootDir.mkdirs()
    }

    fun directory(name: String) {
        val file = File(rootDir, name)
        file.mkdirs()
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
    val a1 = File(a, "a1")
    a1.mkdirs()
    val a2 = File(a, "a2")
    a2.mkdirs()

    val process = Runtime.getRuntime().exec("ls -R ${rootDir.canonicalPath}")
    println(process.inputStream.reader().readLines().joinToString("\n"))
    process.waitFor()
}
```

Puisque nous avons supprimé la ligne `val a = File(rootDir, "a")` il n'est plus possible d'instancier 'a1' et 'a2'. Nous allons
remédier à cela bientôt mais d'abord nous devons faire un petit interlude.

