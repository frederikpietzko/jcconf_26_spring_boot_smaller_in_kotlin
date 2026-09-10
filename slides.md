---
theme: kotlin
transition: view-transition
title: "Why Spring Boot feels smaller in Kotlin"
class: text-center
drawings:
  persist: false
comark: true
duration: 60min
kodee: welcome
highlighter: shiki
---
<!-- @formatter:off -->


[//]: # (---)

[//]: # (name: "Why Spring Boot feels smaller in Kotlin")

[//]: # (layout: intro)

[//]: # (kodee:)

[//]: # (  variant: greeting)

[//]: # (  position: featured)

[//]: # (  size: large)

[//]: # (---)

<div class="mt--15% ml--5% max-w-60%">
<h1 >
Why Spring Boot feels smaller in Kotlin 
</h1>
<p>
Frederik Pietzko
</p>
</div>



---
kodee:
  variant: wink
  position: corner
---

# Why does Spring feel smaller in Kotlin?

<TwoColsVClick>

<template v-slot:left>
<h3 class="mb-5">Kotlin features</h3>

- concise Syntax compare to Java
- Strict Nullability
- Smart Casts
- Extension Functions
- Default Arguments

</template>

<template v-slot:right>
<h3 class="mb-5">for Spring</h3>

- Kotlin Extensions for Spring
- Kotlin DSLs in Spring
- JSpecify
- Compiler Plugins
- Kotlin Ecosystem

</template>

</TwoColsVClick>

<!--
- a lot comes down to the language
- Some language features naturally make Kotlin shorter that Spring
- and there are some Spring specific Extensions that make Spring + Kotlin work better together
-->

---
kodee:
  variant: wink
  position: corner
---

# Concise Syntax

<br />

<DrawnAnnotation type="circle" text="requireNotNull" label="does the null check and throws an IllegalArgumentException with provided message" :on="1" :geometry="{ label: { x: 0.4848, y: 0.5216, width: 0.7395 } }">
<DrawnAnnotation type="underline" text="PetServiceImpl(
    private val repository: PetRepository,
    private val validator: PetValidator,
)" label="concise constructors & properties" :on="3" :geometry="{ label: { x: 0.5035, y: 0.6993 }, connector: { start: { x: 0.3978, y: 0.4599 }, end: { x: 0.4030, y: 0.6774 } } }">

<DrawnAnnotation type="underline" text=": PetService" label="inheritance with `:`" :on="4">

````md magic-move

```java
public Pet getPet(Long id) {
    return repository.findById(id)
        .orElseThrow(() -> 
                new IllegalArgumentException("Pet not found")
        );
}
```

```kotlin
fun getPet(id: Long) = requireNotNull(repository.findByIdOrNull(id)) {
    "Pet not found"
}
```

```java
@Service
class PetServiceImpl implements PetService {
    private final PetRepository repository;
    private final PetValidator validator;
    
    public PetService(PetRepository repository,
                      PetValidator validator) {
        this.repository = repository;
        this.validator = validator;
    }
}
```

```kotlin
@Service
class PetServiceImpl(
    private val repository: PetRepository,
    private val validator: PetValidator,
) : PetService
```

```kotlin
@Service
class PetServiceImpl(
  private val repository: PetRepository,
  private val validator: PetValidator,
) : PetService
```
````
</DrawnAnnotation>
</DrawnAnnotation>
</DrawnAnnotation>

<!--
- For example Kotlin can infer return type
- and throwing an IllegalArgumentException if no Pet is found can be done by `requireNotNull`

- Doing constructor injection is also shorter in Kotlin
- because we can declare properties directly in the constructor
- as well as inheritance, since Kotlin denotes it with `:` rather than `implements`
-->

---
kodee:
  variant: wink
  position: corner
---

# Strict Nullability

<DrawnAnnotation text="final var petName = pet.getName();" label="potentially throws NullPointerException" />
<DrawnAnnotation text="pet.name" label="pet is smart cast to Pet!" :at="2" :until="6"/>
<InlineCompilerError text="pet.name" message="Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable..." :on="1" style="--inline-compiler-error-message-size: .9rem" >

````md magic-move

```java
public Visit scheduleVisit(Long petId, OffsetDateTime at) {
    final var pet = petRepository.findById(petId).orElseNull();
    final var petName = pet.getName(); 
    return visitRepository.save(new Visit(petName, at)) 
}
```

```kotlin
fun scheduleVisit(petId: Long, at: OffsetDateTime): Visit {
    val pet: Pet? = petRepository.findByIdOrNull(petId)
    val petName = pet.name
    return visitRepository.save(Visit(petName, at))
}
```

```kotlin
fun scheduleVisit(petId: Long, at: OffsetDateTime): Visit {
  val pet: Pet? = petRepository.findByIdOrNull(petId) // Type: Pet?
  if (pet == null) throw IllegalArgumentException(
    "Pet with id: $petId not found!"
  )
  val petName = pet.name
  return visitRepository.save(Visit(petName, at))
}
```

```kotlin

fun scheduleVisit(petId: Long, at: OffsetDateTime): Visit {
  val pet: Pet = petRepository.findByIdOrNull(petId) 
      ?: throw IllegalArgumentException(
        "Pet with id: $petId not found!"
      )
  val petName = pet.name 
  return visitRepository.save(Visit(petName, at))
}
```

```kotlin
fun scheduleVisit(petId: Long, at: OffsetDateTime): Visit {
  val pet: Pet? = petRepository.findByIdOrNull(petId)
  requireNotNull(pet) { "Pet with id: $petId not found!" }
  val petName = pet.name
  return visitRepository.save(Visit(petName, at))
}
```

```kotlin
fun scheduleVisit(petId: Long, at: OffsetDateTime): Visit {
  val pet = requireNotNull(petRepository.findByIdOrNull(petId)) {
      "Pet with id: $petId not found!"
  } 
  val petName = pet.name
  return visitRepository.save(Visit(petName, at))
}
```

```kotlin
inline fun <T : Any> requireNotNull(
  value: T?, lazyMessage: () -> Any
): T {
  contract {
    returns() implies (value != null)
  }

  if (value == null) {
    throw IllegalArgumentException(lazyMessage.toString())
  } else {
    return value
  }
}
```

````
</InlineCompilerError>

<!--
- Kotlin is strict about nullability
- Compiler enforces null checks for you
- if you do a null check and and throw an exception
- compiler knows that pet cannot be null afterwards and so smart casts it to Pet without the questionmark
- but since this a very common pattern you can write it differently by using evlis operator
- or using requireNotNull which throws an IllegalArgumentException
- and you can even combine it into one line because requireNotNull returns the argument passed to it
- this works thanks to the Contracts API
- if you are brave try writing contracts yourself, they have some limitations but can be very usefull
-->

---
kodee:
  variant: wink
  position: corner
---

# Smart Casts


<DrawnAnnotation text="is Scheduled -> result.visit" label="result is smart cast to Scheduled" :on="1" />
<DrawnAnnotation text="if(result is Scheduled) return result.visit" label="Data flow based exhaustiveness"  :geometry="{ label: { x: 0.6299, y: 0.4887 } }"/>
<InlineCompilerError text="when" message="when must be exhaustive" :at="4" :until="5">

````md magic-move

```kotlin
val pet = petRepository.findByIdOrNull(petId)
if (pet == null) {
  // pet has type Pet?
} else {
  // pet has type Pet! 
}

```

```kotlin
val stringOrInt: Any = if (Random.nextBoolean()) "string" else 1
if (stringOrInt is String) {
    // stringOrInt is smart cast to String
}
if(stringOrInt is Int) {
    // stringOrInt is smart cast to Int
}
// stingOrInt is Any
```

```kotlin
val stringOrInt: Any = if (Random.nextBoolean()) "string" else 1
when(stringOrInt) {
    is String -> // stringOrInt is smart cast to String
    is Int -> // stringOrInt is smart cast to Int
    else -> // stringOrInt is Any
}
```

```kotlin
sealed interface SchedulingResult {
    data class Scheduled(val visit: Visit): SchedulingResult
    data class SlotTaken(val takenBy: Visit): SchedulingResult
    data object VetOnVacation: SchedulingResult
}

return when(val result = scheduleVisit(...)) {
    is Scheduled -> result.visit
    is SlotTaken -> error("Slot unavailable")
    VetOnVacation -> error("Vet on vacation")
}
```

```kotlin
sealed interface SchedulingResult {
    data class Scheduled(val visit: Visit): SchedulingResult
    data class SlotTaken(val takenBy: Visit): SchedulingResult
    data object VetOnVacation: SchedulingResult
    data object VetSick: SchedulingResult // newly added
}

return when(val result = scheduleVisit(...)) {
    is Scheduled -> result.visit
    is SlotTaken -> error("Slot unavailable")
    VetOnVacation -> error("Vet on vacation")
}
```

```kotlin
sealed interface SchedulingResult {
    data class Scheduled(val visit: Visit): SchedulingResult
    data class SlotTaken(val takenBy: Visit): SchedulingResult
    data object VetOnVacation: SchedulingResult
    data object VetSick: SchedulingResult // newly added
}

if(result is Scheduled) return result.visit

return when(result) {
    is SlotTaken -> error("Slot unavailable")
    VetOnVacation -> error("Vet on vacation")
    VetSick -> error("Vet is sick")
}
```

````

</InlineCompilerError> 

<!--
- I've already shown smart casts a bit
- but they also work for other things than nullability
- but also for types
- so we have a variable of type Any here
- and we can check for the type and it will be smart cast inside of the relevant scope
- and we could also write this using a `when` expression
- which ensures exhausitiveness in the check
- this is particularly useful when we are working with sealed interfaces
- because the compiler will force us to either add an else clause, or even better make the expression exhaustive
- if we enable Data Flow based exhaustiveness checks for when expressions we don't need to handle Scheduled anymore in the expression
-->

---
kodee:
  variant: wink
  position: corner
---

# Extension Functions

<DrawnAnnotation text="fun toDto() = PetDto(id, name)" label="polluted Domain Model" />
<DrawnAnnotation text="private fun Pet.toDto() = PetDto(id, name)" label="Extension Function in Controller" />

````md magic-move

```kotlin
val pet = repository.findByIdOrNull(petId)
```

```kotlin
fun <T: Any, ID: Any> CrudRepository<T, ID>.findByIdOrNull(id: ID): T? =
    findById(id).orElse(null)
```

```kotlin
@Entity
class Pet(
    @Id
    var id: Long? = null,
    var name: String
) {
    fun toDto() = PetDto(id, name)
}
```

```kotlin
package org.example.api

class PetController {
    @GetMapping
    fun getPet(id: Long): PetDto {
        val pet = requireNotNull(repository.findByIdOrNull(id))
        return pet.toDto()
    }

    private fun Pet.toDto() = PetDto(id, name)
}

```
````

<!--
- I've used one function in the previous examples
- findByIdOrNull
- this function is not present in the Repository from Spring by default
- instead it is an extension function implemented by the Spring Team
- Kotlin enables you to extend Classes or interfaces
- this can be usefull to create a more pleasent API for legacy code
- better seperation of concerns
- no pollution of domain model
- or creating ugly UtilClasses
-->

---
kodee:
  variant: wink
  position: corner
---

# Default Arguments

<DrawnAnnotation text="at: OffsetDateTime = OffsetDateTime.now() + Duration.ofDays(2)" :at="0" :until="1" />
<DrawnAnnotation text="at: OffsetDateTime = OffsetDateTime.now() + Duration.ofDays(2)" label="How to use this from Java?" :at="1" :until="2" />
<DrawnAnnotation type="cirlce" text="@JvmOverloads" label="Generates Overloads for Java Callers" />

````md magic-move

```kotlin
fun scheduleVisit(
  petId: Long,
  at: OffsetDateTime = OffsetDateTime.now() + Duration.ofDays(2)
) : Visit {
    //...
}
```

```kotlin
fun scheduleVisit(
  petId: Long,
  at: OffsetDateTime = OffsetDateTime.now() + Duration.ofDays(2)
) : Visit {
  //...
}
```

```kotlin
@JvmOverloads
fun scheduleVisit(
  petId: Long, 
  at: OffsetDateTime = OffsetDateTime.now() + Duration.ofDays(2)
) : Visit {
  //...
}
```

```java
public Visit scheduleVisit(Long petId, OffsetDateTime at) {
    // ...
}

public Visit scheduleVisit(Long petId) {
    return scheduleVisit(
      petId,
      OffsetDateTime.now().plus(Duration.ofDays(2))
    );
}
```

````

<!--
- Kotlin has the ability to provide default arguments
- which enables you to get rid of a lot of function overloading boilerplate
- if you want to call such a function from Java, you should add the `@JvmOverloads` annotation
- which will generate all of the possible overloads for you
- the same is also true for constructors
-->


---
kodee:
  variant: wink
  position: corner
---

# Kotlin specific Extension Functions in Spring

<br />

<DrawnAnnotation type="circle" text="reified T: Any" label="allows access to class of generic" />
<DrawnAnnotation type="circle" text="T::class.java" />

````md magic-move

```kotlin
val pet = petRepository.findByIdOrNull(id)
```

```kotlin
val pet = petRepository.findByIdOrNull(id)

val pet = restTemplate.getForEntity<Pet>("/")
```

```kotlin
val pet = petRepository.findByIdOrNull(id)

val pet = restTemplate.getForEntity<Pet>("/")

val pet = jdbcTemplate.queryForObject<Pet>(...)
```

```kotlin
inline fun <reified T: Any> RestOperations.getForEntity(
  url: String,
  vararg uriVariables: Any?
): ResponseEntity<T> =
  getForEntity(url, T::class.java, *uriVariables)
```

````

<!--
- There are some Kotlin Specific extension functions in Spring
- I've already shown `findByIdOrNull`
- but there are a lot more and I won't show an exclusive List
- but there are `restTemplate.getForEntity` which allows you to pass the entity type as an generic
- or the jdbcTemplate that allows something similar for quering
- Now in java such things are not possible because of type erasure
- and since Kotlin also runs on the JVM it suffers from the same Propblem
- the answer is that Kotlin cheats
- the concept is reified generics
- only possible for `inline` functions, so these aren't real but instead get inlined by the compiler at their call site
- this allows direct access to the class instead of the generic type
-->

---
kodee:
variant: wink
position: corner
---

# JSpecify

- Standard for nullability Annotations
- by Google, JetBrains, Meta, Microsoft, Oracle, Broadcom, ...
- Supported by IntelliJ IDEA, NullAway, Checker Framework
- and Kotlin!
- Spring Framework 7 & Spring Boot 4 completely annotated => seemless Kotlin interop

---
kodee:
  variant: wink
  position: corner
---

# Kotlin DSLs in Spring

````md magic-move

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) {
  return http
      .cors(cors -> cors.disable())
      .csrf(csrf -> csrf.disable())
      .authorizeHttpRequests(auth -> auth
              .requestMatchers("/admin/**").hasRole("ADMIN")
              .anyRequest().permitAll())
      .formLogin(Customizer.withDefaults())
      .httpBasic(Customizer.withDefaults())
      .build();
}
```

```kotlin
@Bean
fun filterChain(http: HttpSecurity): SecurityFilterChain {
  http {
    cors { disable() }
    csrf { disable() }
    authorizeHttpRequests {
        authorize("/admin/**", hasRole("ADMIN"))
        authorize(anyRequest, authenticated)
    }
    formLogin { }
    httpBasic { }
  }
  return http.build()
}
```

```java
@Test
public void testSomething() throws Exception {
  mockMvc.perform(
        MockMvcRequestBuilders.get("/pet/{id}", 1)
            .accept(MediaType.APPLICATION_JSON)
      )
      .andExpect(MockMvcResultMatchers.status().isOk())
      .andExpect(MockMvcResultMatchers.jsonPath("$.id").value(1));
```

```kotlin
@Test
fun `test something`() {
  mockMvc.get("/pet/{id}", 1) {
    accept = APPLICATION_JSON
  }.andExpect {
    status { isOk() }
    jsonPath("$.id") {
      value(1)
    }
  }
}
```

```java
class Registrar implements BeanRegistrar {
  @Override
  public void register(BeanRegistry registry, Environment env) {
    registry.registerBean(PetService.class);
    if (env.matchesProfiles("dev")) {
      registry.registerBean(TestingController.class);
    }
  }
}
@Configuration
@Import(Registrar.class)
class Config {}
```

```kotlin
class Registrar : BeanRegistrarDsl({
  registerBean<PetService>()
  if (env.matchesProfiles("dev")) {
    registerBean<TestingController>()
  }
})

@Configration
@Import(Registrar::class)
class Config
```

```java
class Registrar implements BeanRegistrar {
  @Override
  public void register(BeanRegistry registry, Environment env) {
    registry.registerBean(PetService.class);
    if (env.matchesProfiles("dev")) {
      registry.registerBean(TestingController.class,
        (c) -> c.supplier(context ->
          new TestingController(context.bean(PetService.class)
          )
        )
      );
    }
  }
}
```

```kotlin
class Registrar : BeanRegistrarDsl({
  registerBean<PetService>()
  if (env.matchesProfiles("dev")) {
    registerBean { TestingController(bean()) }
  }
})
```

```kotlin
registerBean {
  val http = bean<HttpSecurity>()
  http {
    cors { disable() }
    csrf { disable() }
    authorizeHttpRequests {
      authorize("/admin/**", hasRole("ADMIN"))
      authorize(anyRequest, authenticated)
    }
    formLogin { }
    httpBasic { }
  }
  http.build()
}
```

````

<!--
- Kotlin is also famous for its DSLs
- and there are a bunch of them in Spring, created by the Spring Team!
- for example the Spring security DSL that allows you to configure it declaritvely

- and a dsl for mockmvc that is a lot easier to read imho

- one of my favorites is the new Spring Registrar in Spring Boot 4
- maybe I'm weird but I've encountered multiple occasions where I needed to programatically register beans

- in Kotlin we have the Spring BeanRegistrarDsl which we can use to programatically do the same
- but a bit more readable

- but when we want to customize bean registration and provide a factory function in java
- it gets a quite ugly quickly

- while in kotlin thanks to type inference we can something like this which is functionally equivalant

- we can also combine the BeanRegistrarDsl with the Spring Security Dsl like so
-->

---

# Router DSL

<style>
.router-dsl-scroll {
  max-height: 40vh;
  overflow-y: auto;
  border-radius: .5rem;
}
.router-dsl-scroll .slidev-code-wrapper,
.router-dsl-scroll .slidev-code {
  max-height: none;
}
</style>

<div class="router-dsl-scroll">

````md magic-move

```kotlin
fun petHandler(petService: PetService) = router {
  "/pets".nest {
    GET {
      val pets = petService.getAllPets()
      ok().body(pets)
    }
    POST {
      val pet = petService.createPet(it.body<PetDto>())
      ok().body(pet)
    }
    "/{id}".nest {
      GET {
        val pet = petService.getById(it.pathVariable("id").toLong())
        ok().body(pet)
      }
      DELETE {
        petService.delete(it.pathVariable("id").toLong())
        noContent().build()
      }
    }
  }
}
```

```kotlin
class Registrar : BeanRegistrarDsl({
    registerBean<PetSerivce>()
    registerBean { petHandler(bean()) }
})

@Configuration
@EnableAutoConfiguration
@Import(Registrar::class)
class Application
```

````

</div>

<!--
- one more DSL I want to show you
- which is the Spring Router DSL
- which allows you to turn Spring into a micro framework
- and do functional routing
- I will not go more in depth here
- but you can register it using the BeanRegistrarDsl as well
- you can even go as far as stop using Class Path Scanning and register all beans manually like this
- and use functional routing
- provided you don't use SpringDataJpa & Hibernate this will drastically improve your startup time even without Spring AOT
- when should you use this?
-->

---
kodee:
  variant: wink
  position: corner
---

# When should you use these DSLs?

- Small to medium microservices
- startup time important
- you like functional style programming & apis
- always use test DSLs

---
kodee:
  variant: wink
  position: corner
---

# What makes Kotlin & Spring interop comfortable?

<SpringInteropDiagram class="mt-6" />

<!--
- There is one thing that causes friction for Kotlin + Spring interop

[click] final! classes, methods and properties are final by default in Kotlin

[click] but Spring creates CGLIB proxies for things like @Transactional or @Configuration

[click] and a CGLIB proxy has to subclass your class - which it can't, if the class is final

[click] it would be very inconvenient to use the `open` keyword everywhere, so instead use the `kotlin("plugin.spring")` plugin

[click] it automatically configures the allOpen plugin to open classes & methods annotated with Spring annotations
-->

---
kodee:
  variant: wink
  position: corner
---

# Spring Compiler Plugin

````md magic-move

```kotlin
@Service
open class PetService {
    @Transactional
    open fun scheduleVisit(
      petId: Long,
      at: OffsetDateTime = OffsetDateTime.now() + Duration.ofDays(2)
    ): Visit {
        ...
    }
}
```

```kotlin
@Service
class PetService {
    @Transactional
    fun scheduleVisit(
      petId: Long,
      at: OffsetDateTime = OffsetDateTime.now() + Duration.ofDays(2)
    ): Visit {
        ...
    }
}
```

````

<!--
- here as an example what you would need to do without the plugin
- and then with the plugin
- you don't need to remember to add the `open` keyword everywhere
-->

---
kodee:
  variant: wink
  position: corner
---

# JPA Compiler Plugin

```kotlin
plugin {
    kotlin("plugin.jpa")
}
```

---
kodee:
  variant: wink
  position: corner
---

# JPA Compiler Plugin


<DrawnAnnotation type="circle" text="()" label="NoArg constructore for Hibernate"  :geometry="{ label: { x: 0.6787, y: 0.3447 } }"/>
<DrawnAnnotation type="circle" text="open" label="added for proper subclassing by hibernate" occurrence="1"  :geometry="{ label: { x: 0.5253, y: 0.2051 } }"/>
<DrawnAnnotation type="circle" text="class Pet" label="NoArg constructore generated & class opened" :on="1" />

````md magic-move

```kotlin
@Entity
open class Pet() {
  @Id
  open var id: Long? = null
  open var name: String? = null

  constructor(
    id: Long? = null,
    name: String? = null,
  ) : this() {
    this.id = id
    this.name = name
  }
}
```

```kotlin
@Entity
class Pet(
    @Id
    var id: Long? = null,
    var name: String,
)
```

````

<!--
- The same is true for JPA
- JPA requires Entities & their properties to be open 
- in order for it to create proxies
- and it also needs a noArg constructor so we would need this whole dance just for hibernate to have it
- with the jpa plugin we don't need to do any of this
- instead the jpa plugin configures the allOpenPlugin automatically correclty for jpa
- and it also configures the noArg plugin automatically to create noArgs constructors automatically for entities
- these noArg constructors are not accessible in Kotlin code (except via reflection) as they are only ment to be used by hibernate
-->

---
kodee:
  variant: wink
  position: corner
---

# Lombok?

- Kotlin cannot see Lombok's generated declarations
- Historically presented a blocker to Kotlin Adoption
- experimental Kotlin Lombok Plugin in `1.5.20`
- promoted to alpha in `2.3.20`
- runs lombok before `kotlinc`

<br />

```kts gradle
plugins {
    kotlin("plugin.lombok")
}
```

---
kodee:
  variant: heart
  position: corner
---

# Future Improvements for Kotlin Lombok Plugin


<DrawnAnnotation type="circle" text="class Pet" label="misses the Builder" :at="1" :until="2" />
<DrawnAnnotation type="circle" text="@Builder" label="Kotlin Plugin will generate Builder" :at="2" :until="3"  :geometry="{ label: { x: 0.5372, y: 0.2103 } }"/>

````md magic-move


```java
@Builder
@AllArgsConstrcutor
@Setter
@Getter
class Pet {
    private Long id;
    private String name;
    private Breed race;
    
    enum Breed {
        CAT, DOG, HORSE,
    }
}
```

```kotlin
class Pet(
    var id: Long,
    var name: String,
    var race: Breed,
) {
    enum class Breed {
        CAT, DOG, HORSE,
    }
}
```

```kotlin
@Builder
class Pet(
  var id: Long,
  var name: String,
  var race: Breed,
) {
  enum class Breed {
    CAT, DOG, HORSE,
  }
}
```
````

<!--
- consider this Pet class annotated with a bunch of Lombok annotations
- some of these like Getter, Setter and AllArgsConstructor map cleanly to Kotlin
- but builder does not
- this presents a barrier for conversion to Kotlin if there are many callers of the Builder
- that's why we are bringing improvements to the Kotlin Lombok Plugin
- and will support generating Kotlin declarations for some of Lombok's annotations
- for example the `@Builder`
-->

---
kodee:
  variant: heart
  position: corner
---

# Future Improvements for Kotlin Lombok Plugin

<VClicks>

- `@Builder`
- `@Log`, `@Slf4j`, ...
- `@NoArgsConstructor`
- `@EqualsAndHashCode`
- `@ToString`
- will probably arrive in `2.5.x`

</VClicks>

<!--
- we will support `@Builder`
- the logging annotations like `@Log`, etc
- `@NoArgsConstructor`, even though that could be easily replicated by configuring the noArg plugin
- `@EqualsAndHashCode`, because its semantics are different in some cases then data classes
- `@ToString`, also because its semantics are different then data classes
- will probably arrive in `2.5.0` maybe a bit later
-->


---
kodee:
  variant: drinking
  position: corner
---

# Kotlin Ecosystem

- Mappie
- MockK
- Kotest
- Exposed

<!--
- one of the things that makes Kotlin great for the Backend is it's ecosystem
- you can rely on the whole existing JVM Ecosystem for free
- with ever improving interop because of JSpecify
- but Kotlin has it's own ecosystem and hase some very cool libraries
- and I want to quickly show 4 of them to you
-->

---
kodee:
  variant: drinking
  position: corner
---

# Mappie

- alternative to MapStruct
- Compiler Plugin
- very good error messages

<br />

````md magic-move

```kotlin
object PetMapper : ObjectMappie<Pet, PetDto> 
```

```kotlin
Target PetDto::id automatically resolved from Pet::id 
but cannot assign source type Long? to target type Long
```

```kotlin
object PetMapper : ObjectMappie<Pet, PetDto> {
  override fun map(from: Pet): PetDto = mapping {
      PetDto::id fromPropertyNotNull from::id
  }
}
```

````

---
kodee:
  variant: drinking
  position: corner
---

# MockK

- excellent mocking library for Kotlin
- Kotlin DSL
- Spring Support through `SpringMockK`

<br />

````md magic-move

```kotlin
val pet = mockk<Pet>()
every { pet.name } returns "Fido"
```

```kotlin
@SpringBootTest
class PetControllerTest {
  @MockkBean
  private lateinit var petService: PetService
  
  @Test
  fun `should return pet`() {
    every { petService.getPet(1L) } returns Pet(1L, "Fido")
  }
}
```

````

<!--
- Mockk is an excellent mocking library for Kotlin
- with a Kotlin DSL to configure your mocks
- and through the SpringMockK has Spring Support
-->

---
kodee:
  variant: drinking
  position: corner
---

# Kotest

- Test Framework with multiple styles
- assertions
- property testing

---
kodee:
variant: drinking
position: corner
---

# Kotest

````md magic-move

```kotlin
class MyTests : FunSpec({
  test("String length should return the length of the string") {
    "sammy".length shouldBe 5
    "".length shouldBe 0
  }
})
```

```kotlin
class MyTests : ShouldSpec({
  should("return the length of the string") {
    "sammy".length shouldBe 5
    "".length shouldBe 0
  }
})
```

```kotlin
class MyTests : BehaviourSpec({
    given("a string") {
        `when`("i measure it's length") {
            then("it should have length of 5") {
                "sammy".length shouldBe 5
            }
        }
    }
})
```

```kotlin
@Test
fun `length of string`() {
    "sammy".length shouldBe 5
}
```

````

<!--
- it provides a lot of different testing styles
- like a functional style FunSpec inspired by Scala
- ShouldSpec directly from the Kotest Team
- BehaviourSpec inspried by Gherkin and other BDD frameworks
- and a bunch of others
- it also provides standalone assertions that can also be used in JUnit
-->

---
kodee:
  variant: drinking
  position: corner
---

# Exposed by JetBrains

- Kotlin DB library with ORM & query DSL flavours
- JDBC & R2DBC Support
- Generate Flyway migrations from Table definitions
- Spring Boot integration


---
kodee:
variant: drinking
position: corner
---

# Exposed by JetBrains



<DrawnAnnotation type="circle" text="LongIdTable" label="Provides Id Column" />
<DrawnAnnotation type="circle" text="varchar" label="Typesafe column definition" :geometry="{ label: { x: 0.7214, y: 0.3385 } }"/>
<DrawnAnnotation type="circle" text="enumeration" label="built in enumeration support"  :geometry="{ label: { x: 0.5973, y: 0.6284 } }"/>
<DrawnAnnotation type="underline" text="PetTable.id eq id" label="typesafe comparisons" :geometry="{ label: { x: 0.6760, y: 0.3148 }, connector: { start: { x: 0.3434, y: 0.3319 }, end: { x: 0.5130, y: 0.3201 } } }"/>
<DrawnAnnotation type="underline" text="this[PetTable.id]" label="typesafe access to columns" />
<DrawnAnnotation type="underline" text="LongEntity(id)" label="DAO definition"  :geometry="{ label: { x: 0.7216, y: 0.0554 } }"/>
<DrawnAnnotation type="underline" text="LongEntityClass" label="Provides CRUD"  :geometry="{ label: { x: 0.6786, y: 0.4173 } }"/>
<DrawnAnnotation type="underline" text="val name by PetTable.name" label="Describes how to resolve entity field from Table" :geometry="{ label: { x: 0.5683, y: 0.6619 } }"/>
<DrawnAnnotation type="underline" text="PetEntity.new" label="creates and saves new PetEntity" :geometry="{ label: { x: 0.6485, y: 0.4206 } }"/>
<DrawnAnnotation type="underline" text="findById" label="also provided by companion" :geometry="{ label: { x: 0.4784, y: 0.4978 } }"/>

````md magic-move

```kotlin
object PetTable : LongIdTable() {
    val name = varchar("name", 50)
    val age = integer("age")
    val breed = enumeration<Breed>("breed")
}

data class Pet(
    val id: Long,
    val name: String,
    val age: Int,
    val breed: Breed,
)
```

```kotlin
@Transactional
fun getPetById(id: Long) = PetTable.selectAll().where {
    PetTable.id eq id
  }.firstOrNull()?.toPet()

fun ResultRow().toPet() = Pet(
    id = this[PetTable.id],
    name = this[PetTable.name],
    age = this[PetTable.age],
    breed = this[PetTable.breed],
)
```

```kotlin
class PetEntity(id: EntityID<Long>) : LongEntity(id) {
    companion object : LongEntityClass<PetEntity>(PetTable)
    val name by PetTable.name
    val age by PetTable.age
    val breed by PetTable.breed
}

```

```kotlin
fun createPet() {
    PetEntity.new {
        name = "Loki"
        age = 10
        breed = Breed.CAT
    }
}
fun findPetById(petId: Long): PetEntity =  PetEntity.findById(petId)
```

````

<!--
- exposed is a db library by JetBrains
- it offers 2 styles of querying: DSL and DAO
- it also offers generation of FlyWay schemas through maven and gradle plugins
- You start by defining a table and your domain object
- and it offers a query DSL that looks a lot like SQL
- But you need to map the ResultRow into your domain object yourself
- although you also have typesafe accessors for that
- it also offers a more ORM like approach if you like that
- by defining an Entity. This will provide the default CRUD operations
-->

---
layout: intro
kodee:
  variant: wave
  size: large
  position: featured
---

# Q & A

Slides

<img class="rounded" src="/qr.png" />
