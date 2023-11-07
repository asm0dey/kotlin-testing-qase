---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: true
# persist drawings in exports and build
drawings:
  persist: false
# page transition
# transition: fade
# use UnoCSS
css: unocss
colorSchema: 'dark'
background: black
download: "https://asm0dey.github.io/kotlin-goodies-ntd/index.pdf"
---

# Kotlin Goodies for Testing

## Pasha Finkelshteyn, <logos-jetbrains />

---
layout: two-cols
---

# `whoami`

- <logos-kotlin-icon /><logos-java /><logos-scala /><logos-clojure/> developer
- Average <logos-python /> enjoyer
- Developer <emojione-avocado /> at <logos-jetbrains />
- Data engineer
- Speaker <twemoji-microphone /> and streamer <logos-twitch /> ([https://twitch.tv/jetbrains](https://jb.gg/twitch) or [<logos-twitch />.<emojione-television />/<logos-jetbrains />](https://jb.gg/twitch))

::right::

![](/avatar.jpg)

---
layout: image
image: /glitched-slide.webp
---


---
layout: center
---

# My promise

During this talk you will find *something* new

<v-clicks>

* Kotlin
* Techniques
* Tools
* Libraries
* Ideas
  
</v-clicks>


---
layout: center
---

# How do we make out tests better?

1. Using DSLs (domain specific languages)
2. Structuring our code better
3. Using smart libraries


---
layout: image-right
image: /bdd.png
---

# BDD

1. Given
2. When
3. Then

---
clicks: 3
---

# Cucumber

```gherkin {all|5}
Feature: An example
  Scenario: The example
    Given I have a username "pasha"
    When I type username in field "x"
    And Press OK button
    Then I get an error "Enter password"
```

<div v-click-hide="1">

What's the problem here?

</div>
<div v-click-hide="2">
<div v-click="1">

This line might intersect with **any** test

</div>
</div>
<div v-click="2">

Actually __**any**__ line can

</div>
<div v-click="3">

Steps match by __name__ O_o:

```java
@And("Press OK button")
void press_ok_button(){/*TODO*/}
```

</div>

---
layout: image-right
image: /kotestbdd.png
---

## How can we do it in Kotlin?

Kotest: https://kotest.io
<br/>

- No obscure text files
- No string matching
- everything in one place

```kotlin {all|1|2|3,8|4,9|5,10|all}
class MyTests : BehaviorSpec({
    given("a broomstick") {
        `when`("I sit on it") {
            then("I should be able to fly") {
                // test code
            }
        }
        `when`("I throw it away") {
            then("it should come back") {
                // test code
            }
        }
    }
})
```

---

## Another flavor of Kotest

Kotest: https://kotest.io
<br/>


```kotlin {all|2|3|4-6|8-9}
class MyTests : DescribeSpec({
    describe("score") {
        it("start as zero") { /* test here */ }
        describe("with a strike") {
            it("adds ten") { /* test here */ }
            it("carries strike to the next frame") { /* test here */ }
        }
        describe("for the opposite team") {
            it("Should negate one score") { /* test here */ }
        }
    }
})
```

---

# Migrating from JUnit?

```kotlin {all|1|2|2-5|6-9|all}
class AnnotationSpecExample : AnnotationSpec() {
    @BeforeEach
    fun beforeTest() {
        println("Before each test")
    }
    @Test
    fun test1() {
        1 shouldBe 1
    }
    @Test
    fun test2() {
        3 shouldBe 3
    }
}
```

---

# 50 shades of kotest

https://kotest.io

|                	|                 	|
|---------------:	|-----------------	|
|       Fun spec 	| String Spec     	|
|    Should spec 	| Describe spec   	|
| Behaviour spec 	| Word spec       	|
|      Free spec 	| Feature spec    	|
|    Expect spec 	| Annotation spec 	|


---
layout: statement
---

# Demo

---

# How do I run the same tests with different inputs?

## Data-driven testing a.k.a Parametrized tests


---

# Parametrized tests? Java

```java {all|7-11|8|1-6|1|2|3|2-4|all}
static Stream<Arguments> stringIntAndListProvider() {
    return Stream.of(
        arguments("apple", 1, Arrays.asList("a", "b")),
        arguments("lemon", 2, Arrays.asList("x", "y"))
    );
} 
@ParameterizedTest
@MethodSource("stringIntAndListProvider")
void testWithMultiArgMethodSource(String str, int num, List<String> list) {/
    * snip */
}
```

<v-click>

`stringIntAndListProvider`, really?

</v-click>
<v-clicks>

- Becomes cluttered
- Not typed (enough)
- Bound by a string!

</v-clicks>

---

# Parametrized tests? Kotlin (with kotest)

```kotlin {all|1|3|4-7|8|9}
fun isPythagTriple(a: Int, b: Int, c: Int): Boolean = a * a + b * b == c * c
// snip…
withData( // ddt
  Triple(3, 4, 5),
  Triple(6, 8, 10),
  Triple(8, 15, 17),
  Triple(7, 24, 25)
) { (a, b, c) -> // destructuring
  isPythagTriple(a, b, c) shouldBe true //test
}
```

---
layout: statement
---

# Demo

---
layout: image
image: /confused.png
---

# But is it enough?

<style>
h1 {
    color: white !important;
    text-align: center;
}
</style>

---
layout: statement
---

# Demo

---

# Time to mock

#### Setup
```kotlin {all|1-3|1|2|5-7|5|6|9}
class UserService(val repo: UserRepository) {
    fun getAllUsers() = repo.allUsers()
}

class UserRepository {
    fun allUsers(): List<User> { TODO("demo") }
}

class User(val name: String, val age: Int)
```

---

# How the test looks in Java

```java {all|1|3,4|6,7|9,10|12,13|15,16}
UserRepository mockRepository = Mockito.mock(UserRepository.class);

// Create some sample user data
List<User> users = Arrays.asList(new User("Alice", 25), new User("Bob", 30), new User("Charlie", 35));

// Define the expected behavior of the mock repository
Mockito.when(mockRepository.allUsers()).thenReturn(users);

// Create an instance of the UserService with the mock repository
UserService userService = new UserService(mockRepository);

// Call the method under test
List<User> result = userService.getAllUsers();

// Verify the result
Assertions.assertEquals(users, result);
```

---

# Time to mock! (MockK edition)

```kotlin {all|1|2|3|5|6}
val users = listOf(new User("Alice", 25), new User("Bob", 30), new User("Charlie", 35))
val repo = mockk<UserRepository> {
    every { allUsers() } returns users
}
val service = UserService(repo)
service.getAllUsers() shouldBe users
```
<div v-click>

Nice DSL

Code describing `UserRepository` is inside `{}`

What if I want to mock users too?

</div>


---

# Time to mock users! (mockk edition)

```kotlin {all|1|2|3-5|6-8|11,12}
val repo = mockk<UserRepository> {
    every { allUsers() } returns listOf(
        mockk {
            every { name } returns "Pasha"
        },
        mockk {
            every { name } returns "Alexandra"
        },
    )
}
val service = UserService(repo)
service.getAllUsers()[1].name shouldBe "Alexandra"
```

Thank Alexandra for helping me with this presentation!

<div v-click>

Users are described inside `User` mocks!

</div>

---

# Atrium expectations!

https://docs.atriumlib.org/

```kotlin {all|1|3|3-4|7-8}
expect(service.getAllUsers()) toContain o inAny order but only the entries(
    {
        its { name } toEqual "Pash"
        its { age } toEqual 36
    },
    {
        its { name } toEqual "Mark"
        its { age } toEqual 13
    }
)
```

<div v-click="2">

I've made a mistake to show the error message

</div>


---

# Error report

```txt {all|1|3|13-14|4-5|all}
I expected subject: [User(#3), User(#4)]        (java.util.Arrays.ArrayList <315506631>)
◆ to contain only, in any order: 
  ✘ an element which needs: 
      » ▶ its.definedIn(MockTest.kt:42): 
          ◾ to equal: "Pash"        <2026086646>
      » ▶ its.definedIn(MockTest.kt:43): 
          ◾ to equal: 36        (kotlin.Int <116289638>)
  ✔ an element which needs: 
      » ▶ its.definedIn(MockTest.kt:46): 
          ◾ to equal: "Mark"        <574921197>
      » ▶ its.definedIn(MockTest.kt:47): 
          ◾ to equal: 13        (kotlin.Int <453296736>)
  ❗❗ following elements were mismatched: 
     ⚬ User(#3)        (User <1041341319>)
```

<div v-click>

- Readable as a text
- Shows the errors

</div>

---

# Dessert 

Now, when we saw different nice DSL-using capabilities…

```kotlin
ideaTest<App>("Create Kafka connection") {
    showBDTPanel {
        createBaseConnection("Kafka") {
            fillCredentialsForKafka(connectionName)
            checkSuccessfulMessage()
            ok()
        }
        checkMonitoringConnection(monitoringToolWindow)
    }
}
```

---
layout: image
image: /magick.png
---

<h1 style="color:white">How does it work? Magick!</h1>

<Link to="39">Dive deeper</Link>

---
layout: image
image: /question.png
---

# What did we learn?

<style>
h1 {
    color: darkorange;
    text-align: center;
}
</style>

---

# What did we learn?

1. Tests should be readable
2. Testing is not "just unit tests" sometimes
3. Kotest gives good structure to the tests.
4. Kotest has built-in property tests
5. Atrium gives us assertions DSL

---
layout: cover
background: /monk.png
---

# Thank you! Time for questions 😄

<div class="text-left">
<p><ph-twitter-logo-fill/> asm0di0</p>
<p><mdi-mastodon/> @asm0dey@fosstodon.org</p>
<p><ph-envelope-simple-open-fill/> me@asm0dey.site</p>
<p><ph-twitch-logo-fill/> https://jb.gg/twitch</p>
<p><ph-github-logo-fill/><ph-instagram-logo-fill/><ph-linkedin-logo-fill/><ph-telegram-logo-fill/> asm0dey</p>
<p><ph-globe-duotone/><a href="https://linktr.ee/asm0dey">https://linktr.ee/asm0dey</a></p>
</div>

---
layout: center
---

<Transform :scale="0.6" origin="center">

![](/presqr.png)

</Transform>

---
layout: end
---

---

# Deeper dive into JetBrains tests

```kotlin
ideaTest<App>("Create Kafka connection") {
```

really means:

```kotlin {all|1|4|6,7}
fun <T : App> Group.ideaTest(title: String, 
                             enabled: Boolean = true, 
                             isMuted: Boolean = false,
                             testBody: T.(ctx: IdeaNodeContext<T>) -> Unit) {
    IdeaNodeContext<T>(scenarioContext).apply {
        test(title, enabled, isMuted) {
            app.testBody(this)
        }
    }
}
```

---

# Deeper dive into JetBrains tests

```kotlin
    showBDTPanel {
```

really means:

```kotlin {all|1|3|5|6|9-11}
fun App.showBDTPanel(block: BDTPanel.() -> Unit) {
    val app = this
    ideaFrame {
        val frame = this
        step("Open Big Data Tools Panel") {
            openBDTPanel(app)
            if (isKafka()) {
                try {
                    container(byXpath("//div[@accessiblename='Kafka Tool Window' and @class='InternalDecoratorImpl']")) {
                        BDTPanel(app, frame, this).block()
                    }
```

---

# Deeper dive into JetBrains tests

```kotlin
        createBaseConnection("Kafka") {
```

really means:

```kotlin {all|1|2|4|5}
fun createBaseConnection(typeOfConnection: String, block: ConnectionDialog.() -> Unit) {
    step("Open connection dialog") {
        if (app.isKafka()) {
            frame.find<ComponentFixture>(byXpath("//div[contains(@tooltiptext.key, 'toolwindow.tab.add')]")).click()
            clickTypeOfConnection(typeOfConnection, block)
            pause(5000)
```

---
layout: center
---

# And this is the abyss!

But you can do the same!

<Link to="35">Go back to summary</Link>

---
layout: end
---
