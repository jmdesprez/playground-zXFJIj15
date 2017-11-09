# Fonctions
Quelques exemples de fonctions en Kotlin : 

```kotlin runnable
// fonction de type 'void'
fun printMessage(message: String) {
    println(message)
}

// fonction qui retourne un entier
fun add(a: Int, b: Int): Int {
    return a + b
}

// une fonction dans une fonction
fun sayHello(friends: List<String>) {
    fun computeMessage(name: String): String {
        return "Hello ${name.capitalize()}!!"
    }

    for (name in friends) {
        val message = computeMessage(name)
        println(message)
    }
}

fun main(args: Array<String>) {
  sayHello(listOf("Aurélie", "Antoine", "Michel"))
}
```

[??!!] il est tout à fait possible (et très pratique) de déclarer des fonctions dans d'autres fonctions. Cela permet de limiter
la visibilité de votre fonction 'interne' (une sorte de 'super private')

De plus, une fonction Kotlin peut avoir des valeurs par défaut pour ses paramètres : 

```kotlin runnable
fun sayHello(name: String = "Alice", greeting: String = "Hello") {
    println("$greeting $name")
}

fun main(args: Array<String>) {
    sayHello()
    sayHello("Aurélie")
    sayHello("Aurélie", "Bonjour")

    // nous pouvons aussi nommer les paramètres
    sayHello(greeting = "Bonjour")
    sayHello(name = "Aurélie")
    sayHello(greeting = "Bonjour", name = "Aurélie")
}
```

# Lambda
Imaginons que vous ayez besoin d'éxecuter un traitement avant et/ou après un code arbitraire (non connu à l'avance).
Une implémentation en JAVA pourrait donner quelque chose dans ce genre : 

```java runnable
public class Main {
    public static void box(final Runnable block) {
        System.out.println("-----------------------------");
        block.run();
        System.out.println("-----------------------------");
    }

    public static void main(String[] args) {
        // Version sans utilisation d'une lambda
        box(new Runnable() {
            @Override
            public void run() {
                System.out.println("Sans Lambda");
            }
        });

        System.out.println("");
        System.out.println("");

        // Version avec utilisation d'une lambda
        box(() -> {
            System.out.println("Avec Lambda");
        });

    }
}
```

On voit que l'utilisation d'une lambda (`() -> {}`) permet de condenser le code et donc d'améliorer la lisibilité.

Il reste cependant un problème car les choses se compliquent si votre bloc de code arbitraire attend des arguments
en entrée. En effet, JAVA ne propose qu'un nombre limité d'options : 

```java
public interface Demo {

    /**
     * Le bloc de code ne nécessite aucun paramètre en entrée ni en sortie.
     * Je peux utiliser Runnable
     */
    void noParam(final Runnable block);

    /**
     * Le bloc de code n'a pas d'entrée mais produit un Integer en sortie.
     * Je peux utiliser Callable
     */
    void out(final Callable<Integer> block);

    /**
     * Le bloc de code prend un String en entrée et produit un Integer en sortie.
     * Je peux utiliser Function
     */
    void inOut(final Function<String, Integer> block);

    /**
     * Le block de code prend deux Integer en entrée et produit un String.
     * Je peux utiliser une BiFunction
     */
    void twoInOut(final BiFunction<Integer, Integer, String> block);

    /**
     * Le block de code prend deux Integer en entrée et ne produit aucune sortie.
     * Je dois quand même utiliser une BiFunction
     */
    void twoInNoOut(final BiFunction<Integer, Integer, Void> block);

    // En JAVA, je ne peux pas aller au dela de deux arguements en entrée et un argument en sortie
}
```

Nous allons voir que Kotlin simplifie grandement les choses (une fois de plus) grace au 
[Function Type](https://kotlinlang.org/docs/reference/lambdas.html#function-types).

Les 'function types' vous permettent de créer des types de fonctions à la volée. Voici l'équivalent Kotlin
de l'interface ci-dessus (avec un rappel de la version JAVA juste à côté) : 

```kotlin
interface Demo {

    /**
     * Le bloc de code ne nécessite aucun paramètre en entrée ni en sortie.
     */
    fun noParam(block: () -> Unit) // JAVA: void noParam(final Runnable block);

    /**
     * Le bloc de code n'a pas d'entrée mais produit un Integer en sortie.
     */
    fun out(block: () -> Int) // JAVA: void out(final Callable<Integer> block);

    /**
     * Le bloc de code prend un String en entrée et produit un Integer en sortie.
     */
    fun inOut(block: (String) -> Int) // JAVA: void inOut(final Function<String, Integer> block);

    /**
     * Le block de code prend deux Integer en entrée et produit un String.
     */
    fun twoInOut(block: (Int, Int) -> String) // JAVA: void twoInOut(final BiFunction<Integer, Integer, String> block);

    /**
     * Le block de code prend deux Integer en entrée et ne produit aucune sortie.
     */
    fun twoInNoOut(block: (Int, Int) -> Unit) // JAVA: void twoInNoOut(final BiFunction<Integer, Integer, Void> block);

    /**
     * Le block de code prend deux Integer et deux String en entrée et ne produit aucune sortie.
     */
    fun fourInNoOut(block: (Int, Int, String, String) -> Unit) // JAVA: Impossible

    /**
     * Il est aussi possible de nommer les paramètres plus plus de lisibilité
     */
    fun namedParameter(userFactory: (id: Int, age: Int, firstname: String, lastname: String) -> User) // JAVA: Impossible
}
```

La syntaxe des 'function types' est en fait très proche de la déclaration standard d'une fonction : 

- `fun block()` est équivalent à `block: () -> Unit`
- `fun add(a: Int, b: Int): Int` est équivalent à `add: (a: Int, b: Int) -> Int` qui est aussi équivalent à `add: (Int, Int) -> Int`

Reprenons l'exemple de la méthode `box` ci-dessus. Voici son équivalent en Kotlin (un peu amélioré) : 
 
```kotlin runnable
fun box(title: String, block: (width: Int) -> Unit) {
    val header = "----------$title----------"
    val width = header.length
    println(header)
    block(width)
    println("-".repeat(width))
}

fun main(args: Array<String>) {
    box("Hello World", {
        // use the witdh to align the message to the right
        println("Left aligned message")
    })
    println()
    println()
    box("Hello World", { width ->
        // use the witdh to align the message to the right
        println("Right aligned message".padStart(width))
    })
}
```

Une lambda Kotlin est déclarée de la façon suivante : 
- on commence par une accolade ouvrante `{`
- puis on ajoute la déclaration des arguments (si besoin) `{ id, firstname, lastname `
- et on termine la déclaration des arguments par une 'flèche' (si besoin) `{ id, firstname, lastname -> `
- ensuite on peut écrire le code que l'on souhaite `{ id, firstname, lastname -> User(id, firstname, lastname)`
- et on termine la déclaration de la lambda par l'accolade fermante `{ id, firstname, lastname -> User(id, firstname, lastname)}`

# Extensions Functions

Attention, nous n'allons pas parler ici d'une extension "standard" (une classe qui hérite d'une autre classe)
mais d'un concept spécifique à Kotlin, les ["fonctions d'extensions"](https://kotlinlang.org/docs/reference/extensions.html#extension-functions).

Nous avons très souvent besoin de méthodes utilitaires sur une classe particulière. Souvent, ces méthodes
se retrouvent dans les classes "Helper" car il n'est pas possible de les intégrer directement à la classe
en question (car la classe est `final` ou le besoin est trop spécifique).

```java runnable
public class Main {

    public static boolean isValid(final String reference) {
        return reference.startsWith("ref-");
    }

    public static void main(final String[] args) {
        if(isValid("ref-12345")) {
            System.out.println("Valid");
        } else {
            System.out.println("Invalid");
        }
    }
}
```

Dans le code ci-dessus, nous aurions préféré pouvoir écrire `if("ref-12345".isValid())` pour une meilleure
lisibilité et pour profiter de l'auto-completion de notre IDE. Malheureusement cela n'est pas possible : 
cette fonction est trop spécifique pour être ajoutée à tous les 'String' et de toute façon cette classe est
`final`.

Kotlin apporte ici une réponse grâce aux fonctions d'extensions : 

```kotlin runnable
fun String.isValid(): Boolean {
    return this.startsWith("ref-")
}

fun main(args: Array<String>) {
    if("ref-12345".isValid()) {
        println("Valid")
    } else {
        println("Invalid")
    }
}
```

Cela peut sembler compliqué mais en réalité, une fonction d'extension est simplement du sucre syntaxique.
La méthode `fun String.isValid(): Boolean` ci-dessus sera compilée comme étant : `fun isValid(receiver: String): Boolean`.
La référence `this` dans la fonction d'extension sera simplement remplacée par `receiver`. Le compilateur
se chargera ensuite de remplacer `"ref-12345".isValid()` par `isValid("ref-12345")` et le tour est joué :)

Puisque les fonctions d'extensions sont avant tout des fonctions, il est tout à fait possible de déclarer
le type correspondant grace aux [Function Type](https://kotlinlang.org/docs/reference/lambdas.html#function-types) : 

|Fonction|Type|
|--------|----|
|fun block()|() -> Unit|
|fun add(a:Int, b: Int)|(a: Int, b: Int) -> Int ou (Int, Int) -> Int|
|fun String.isValid(): Boolean|String.() -> Boolean|
|fun String.trimAt(length: Int): String|String.(length: Int) -> String ou String.(Int) -> String|  

De même, une fonction d'extension peut tout à fait être passée en paramètre d'une autre fonction : 

```kotlin runnable
fun box(block: StringBuilder.() -> Unit): String {
    val builder = StringBuilder()
    builder.append("-------------\n")
    builder.block()   // 2
    builder.append("\n-------------\n")
    return builder.toString()
}

fun main(args: Array<String>) {
    val box = box({  // 1
        this.append("Hello ")  // 1
        append("World")   // 1
    })

    println(box)
}
```

[??!!] Quelques remarques : 
1. Cette lambda implémente le type `StringBuilder.() -> Unit` donc nous sommes dans les mêmes conditions
que lorsque nous faisons une fonction d'extension. Donc `this` correspond au 'receiver' qui est le `StringBuilder`
et nous pouvons donc appeler la méthode `StringBuilder#append`
2. Puisque le type de `block` est une fonction d'extension sur un `StringBuilder`, nous pouvons faire "comme si" la classe
`StringBuilder` possédait la méthode `block` et donc invoquer cette méthode directement : `builder.block()`

# Exercices
N'hésitez pas à faire quelques essais de fonctions d'extensions en vous aidant des exemples ci-dessus.
