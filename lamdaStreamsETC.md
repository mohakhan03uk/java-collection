
# Java Lambda Expressions — Mini‑Lab (GitHub‑Ready)

A hands‑on, **beginner‑to‑advanced** lab for Java lambda expressions, Streams, Collectors, `CompletableFuture`, custom functional interfaces, and patterns for **checked exceptions**. Designed for copy‑paste and quick execution.

> Works on **Java 8+**. Notes call out features that require newer versions (e.g., `var` in lambda params requires **Java 11+**; `Collectors.teeing` requires **Java 12+**).

---

## 1) Prerequisites

- **JDK**: Java 8+ (recommend **Java 17 LTS** or newer)
- **Editor/Build**: Any IDE or plain CLI
- **CLI compile/run** (no build tool):
  ```bash
  # Create a workspace
  mkdir -p lab && cd lab
  # Save code snippets as .java files in this folder
  javac *.java && java Main
  ```
- **Concepts to recall** (one‑liners):
  - *Functional interface*: an interface with **one** abstract method (SAM). Examples: `Function`, `Predicate`.
  - *Lambda*: concise implementation of a functional interface: `x -> x + 1`.
  - *Method reference*: reuse existing methods: `System.out::println`, `String::trim`.
  - *Effectively final*: captured local variables must not be reassigned after capture.

---

## 2) Why Lambdas (Quick Primer)

- Reduce **boilerplate** (vs anonymous classes)
- Enable **Streams** (`map`, `filter`, `reduce`) and functional style
- Compose **callbacks** and async pipelines (`CompletableFuture`)
- Improve readability and expressiveness for small behaviors

---

## 3) Syntax Cheat Sheet

```java
() -> 42                      // no params, returns 42
x -> x * x                    // one param, inferred type
(int a, int b) -> a + b       // explicit typed params
(x, y) -> { int s = x + y; return s; } // block body

// Method references
System.out::println           // Consumer<String>
String::length                // Function<String,Integer>
new ArrayList<>()             // Supplier<List<?>> (via constructor ref: ArrayList::new)
```

**Core functional interfaces (java.util.function):**
- `Supplier<T>  // T get()`
- `Consumer<T>  // void accept(T t)`
- `Function<T,R>  // R apply(T t)`
- `Predicate<T>  // boolean test(T t)`
- `UnaryOperator<T>  // T apply(T t)`
- `BinaryOperator<T> // T apply(T t, T u)`
- `BiFunction<T,U,R>`, `BiConsumer<T,U>`, `BiPredicate<T,U>`

---

## 4) Variable Capture & `this`

- Locals captured by a lambda must be **effectively final**.
- Lambdas capture **by value**; you cannot mutate captured locals.
- Inside a lambda, `this` refers to the **enclosing instance** (unlike anonymous classes, where `this` is the anonymous class itself).

---

## 5) Streams in One Picture

```mermaid
flowchart LR
  A[Collection] --> B[stream]
  B --> C[filter]
  C --> D[map]
  D --> E[collect/reduce]
  E --> F[Result]
```

- Streams are **lazy** until a **terminal** operation (e.g., `collect`, `forEach`, `reduce`).
- Prefer **primitive streams** (`IntStream`, `LongStream`, `DoubleStream`) to avoid boxing overhead.

---

## 6) Collectors You’ll Use A Lot

- `Collectors.toList()` / `.toSet()`
- `Collectors.joining(",")`
- `Collectors.groupingBy(keyFn, downstream)`
- `Collectors.partitioningBy(pred)`
- `Collectors.mapping(mapper, downstream)`
- `Collectors.reducing(...)`
- *(Java 12+)* `Collectors.teeing(downstream1, downstream2, merger)`

---

## 7) CompletableFuture Essentials

```java
CompletableFuture.supplyAsync(() -> fetch())
    .thenApply(this::transform)
    .thenCompose(this::saveAsync) // flat-map to another future
    .thenAccept(id -> System.out.println("Saved " + id))
    .exceptionally(ex -> { ex.printStackTrace(); return null; });
```
- Use `thenCompose` (not `thenApply`) when the function returns a `CompletableFuture`.
- Combine tasks: `thenCombine`, `allOf`, `anyOf`.

---

## 8) Checked Exceptions Patterns

Standard functional interfaces **don’t declare checked exceptions**. Options:
1) **Wrap** checked exceptions as unchecked (e.g., `UncheckedIOException`).
2) Define your **own** functional interfaces with `throws`.
3) Write a small **adapter** that rethrows as unchecked.

Examples are in the exercises and solutions.

---

## 9) Hard Bits Demystified

- **Boxing**: `Stream<Integer>` is slower than `IntStream`. Prefer primitives in tight loops.
- **Parallel streams**: great for large CPU‑bound, stateless tasks; harmful for small or IO‑bound work.
- **Stateful lambdas**: avoid shared mutable state (esp. with `parallel()`): prefer pure functions and use reductions/collectors.
- **Ordering**: `unordered()` can speed some operations; but be careful if you rely on encounter order.
- **Identity semantics**: don’t rely on `equals`/`hashCode` of lambda instances.

---

# 10) Exercises

> Put tasks into `Main.java` (or separate files). Each exercise lists a **goal** and a **hint**. Solutions follow later.

## Exercise 1 — Warm‑up: map/filter
**Goal**: Given `List<String> names = List.of("alice","bob","charlie","al" );` create a new list of **uppercase** names with length > 3, sorted ascending.

**Hint**: `stream()`, `filter`, `map`, `sorted`, `toList()` (Java 16+) or `collect(Collectors.toList())`.

---

## Exercise 2 — Method references & Comparator
**Goal**: Sort a list of `Person(firstName,lastName,age)` by lastName, then firstName, ascending.

**Hint**: `Comparator.comparing(Person::getLastName).thenComparing(Person::getFirstName)`.

---

## Exercise 3 — Grouping & Counting
**Goal**: From a list of words, produce a `Map<Character, Long>` counting words by their **first letter**. Ensure letters are treated case‑insensitively.

**Hint**: `groupingBy(keyFn, counting())`; normalize with `toLowerCase()`.

---

## Exercise 4 — Primitive streams
**Goal**: Compute the sum of squares of **even** numbers in `[1, 1_000_000]` using `IntStream`.

**Hint**: `IntStream.rangeClosed(1, 1_000_000).filter(...).map(...).sum()`.

---

## Exercise 5 — CompletableFuture chain
**Goal**: Simulate an async pipeline: `fetch()` -> `parse()` -> `saveAsync()` -> print ID, with error handling.

**Hint**: `supplyAsync`, `thenApply`, `thenCompose`, `thenAccept`, `exceptionally`.

---

## Exercise 6 — Checked exceptions in Streams
**Goal**: Read multiple small text files into a single String (joined by `\n`), ignoring files that do not exist. Use a **safe wrapper** so you can call `Files.readString(Path)` inside a stream without compilation errors.

**Hint**: Write `static <T,R> Function<T,R> unchecked(ThrowingFunction<T,R,? extends Exception> f)` that wraps checked exceptions.

---

## Exercise 7 — Custom Functional Interface
**Goal**: Create `@FunctionalInterface interface IOFunction<T,R> { R apply(T t) throws IOException; }` and use it in a reusable `static <T,R> Function<T,R> uncheck(IOFunction<T,R> f)` adapter.

**Hint**: Implement by catching `IOException` and throwing `UncheckedIOException`.

---

## Exercise 8 — Custom Collector (hard)
**Goal**: Implement a custom `Collector<Integer, Stats, Stats>` that computes `count`, `sum`, `min`, `max`, and `average` in **one pass**.

**Hint**: Use `Collector.of(supplier, accumulator, combiner, Characteristics.UNORDERED)` and a mutable `Stats` container.

---

## Exercise 9 — Parallel stream pitfall (hard)
**Goal**: Show that mutating a shared `ArrayList<Integer>` inside a `parallelStream().forEach(...)` can lose or corrupt data. Then fix it using a proper **collector**.

**Hint**: Reproduce the issue, then replace `forEach` + shared list with `collect(toList())` or `mapToInt(...).sum()`.

---

# 11) Starter Code (Main.java)

```java
import java.io.*;
import java.nio.file.*;
import java.util.*;
import java.util.concurrent.*;
import java.util.function.*;
import java.util.stream.*;

public class Main {
    // ----- Exercise 5 helpers -----
    static String fetch() {
        sleep(100); // simulate IO
        return "{id:42,name:alice}";
    }
    static Map<String,String> parse(String json) {
        sleep(50);
        return Map.of("id", "42", "name", "alice");
    }
    static CompletableFuture<String> saveAsync(Map<String,String> m) {
        return CompletableFuture.supplyAsync(() -> {
            sleep(80);
            return "ID-" + m.get("id");
        });
    }
    static void sleep(long ms) { try { Thread.sleep(ms); } catch (InterruptedException ignored) {} }

    // ----- Exercise 2 model -----
    static class Person {
        private final String firstName; private final String lastName; private final int age;
        Person(String f, String l, int a) { this.firstName=f; this.lastName=l; this.age=a; }
        String getFirstName(){return firstName;} String getLastName(){return lastName;} int getAge(){return age;}
        public String toString(){return firstName+" "+lastName+"("+age+")";}
    }

    // ----- Exercise 6/7 helpers -----
    @FunctionalInterface
    interface ThrowingFunction<T,R,E extends Exception> { R apply(T t) throws E; }

    static <T,R> Function<T,R> unchecked(ThrowingFunction<T,R,? extends Exception> f) {
        return t -> {
            try { return f.apply(t); }
            catch (RuntimeException e) { throw e; }
            catch (Exception e) { throw new RuntimeException(e); }
        };
    }

    @FunctionalInterface
    interface IOFunction<T,R> { R apply(T t) throws IOException; }

    static <T,R> Function<T,R> uncheck(IOFunction<T,R> f) {
        return t -> {
            try { return f.apply(t); }
            catch (IOException e) { throw new UncheckedIOException(e); }
        };
    }

    // ----- Exercise 8 helper -----
    static class Stats {
        long count; long sum; int min = Integer.MAX_VALUE; int max = Integer.MIN_VALUE;
        void add(int x){ count++; sum+=x; min=Math.min(min,x); max=Math.max(max,x); }
        Stats combine(Stats other){
            Stats s = new Stats();
            s.count = this.count + other.count;
            s.sum = this.sum + other.sum;
            s.min = Math.min(this.min, other.min);
            s.max = Math.max(this.max, other.max);
            return s;
        }
        double avg(){ return count==0? 0.0 : (double) sum / count; }
        public String toString(){ return String.format("Stats{count=%d,sum=%d,min=%d,max=%d,avg=%.2f}", count, sum, min, max, avg()); }
    }

    public static void main(String[] args) throws Exception {
        // You can progressively fill solutions here or run the Solutions class separately.
        System.out.println("Java Lambda Mini‑Lab — fill in exercises or run Solutions.main()");
    }
}
```

---

# 12) Solutions (copy into `Solutions.java` and run)

```java
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.*;
import java.util.concurrent.*;
import java.util.function.*;
import java.util.stream.*;

public class Solutions {

    // Reuse models and helpers by copy‑pasting from Main or keeping both in one file.
    static void sleep(long ms) { try { Thread.sleep(ms); } catch (InterruptedException ignored) {} }

    static class Person {
        private final String firstName; private final String lastName; private final int age;
        Person(String f, String l, int a) { this.firstName=f; this.lastName=l; this.age=a; }
        String getFirstName(){return firstName;} String getLastName(){return lastName;} int getAge(){return age;}
        public String toString(){return firstName+" "+lastName+"("+age+")";}
    }

    @FunctionalInterface
    interface ThrowingFunction<T,R,E extends Exception> { R apply(T t) throws E; }

    static <T,R> Function<T,R> unchecked(ThrowingFunction<T,R,? extends Exception> f) {
        return t -> {
            try { return f.apply(t); }
            catch (RuntimeException e) { throw e; }
            catch (Exception e) { throw new RuntimeException(e); }
        };
    }

    @FunctionalInterface
    interface IOFunction<T,R> { R apply(T t) throws IOException; }

    static <T,R> Function<T,R> uncheck(IOFunction<T,R> f) {
        return t -> {
            try { return f.apply(t); }
            catch (IOException e) { throw new UncheckedIOException(e); }
        };
    }

    static class Stats {
        long count; long sum; int min = Integer.MAX_VALUE; int max = Integer.MIN_VALUE;
        void add(int x){ count++; sum+=x; min=Math.min(min,x); max=Math.max(max,x); }
        Stats combine(Stats other){
            Stats s = new Stats();
            s.count = this.count + other.count;
            s.sum = this.sum + other.sum;
            s.min = Math.min(this.min, other.min);
            s.max = Math.max(this.max, other.max);
            return s;
        }
        double avg(){ return count==0? 0.0 : (double) sum / count; }
        public String toString(){ return String.format("Stats{count=%d,sum=%d,min=%d,max=%d,avg=%.2f}", count, sum, min, max, avg()); }
    }

    public static void main(String[] args) throws Exception {
        exercise1();
        exercise2();
        exercise3();
        exercise4();
        exercise5();
        exercise6();
        exercise7();
        exercise8();
        exercise9();
    }

    static void exercise1() {
        List<String> names = List.of("alice","bob","charlie","al");
        List<String> res = names.stream()
                .filter(n -> n.length() > 3)
                .map(String::toUpperCase)
                .sorted()
                .toList(); // use collect(Collectors.toList()) on Java < 16
        System.out.println("Ex1: " + res);
    }

    static void exercise2() {
        List<Person> people = List.of(
                new Person("Alice","Zephyr",30),
                new Person("Bob","Yellow",22),
                new Person("Alice","Yellow",28)
        );
        List<Person> sorted = new ArrayList<>(people);
        sorted.sort(Comparator.comparing(Person::getLastName)
                              .thenComparing(Person::getFirstName));
        System.out.println("Ex2: " + sorted);
    }

    static void exercise3() {
        List<String> words = List.of("Apple","ape","Bee","bob","car","Car");
        Map<Character, Long> counts = words.stream()
                .collect(Collectors.groupingBy(w -> Character.toLowerCase(w.charAt(0)),
                        Collectors.counting()));
        System.out.println("Ex3: " + counts);
    }

    static void exercise4() {
        int sum = java.util.stream.IntStream.rangeClosed(1, 1_000_000)
                .filter(x -> x % 2 == 0)
                .map(x -> x * x)
                .sum();
        System.out.println("Ex4: sum=" + sum);
    }

    static void exercise5() {
        CompletableFuture.supplyAsync(() -> "{id:42,name:alice}")
                .thenApply(json -> Map.of("id","42","name","alice"))
                .thenCompose(map -> CompletableFuture.supplyAsync(() -> "ID-" + map.get("id")))
                .thenAccept(id -> System.out.println("Ex5: Saved " + id))
                .exceptionally(ex -> { System.err.println("Ex5 error: "+ex); return null; })
                .join();
    }

    static void exercise6() throws Exception {
        // Prepare demo files
        Path a = Files.writeString(Files.createTempFile("a", ".txt"), "hello A", StandardCharsets.UTF_8);
        Path b = Files.writeString(Files.createTempFile("b", ".txt"), "hello B", StandardCharsets.UTF_8);
        Path missing = Path.of("/path/does/not/exist.txt");

        String joined = Stream.of(a, b, missing)
                .filter(Files::exists)
                .map(uncheck(Files::readString))
                .collect(Collectors.joining("\n"));
        System.out.println("Ex6: \n" + joined);
    }

    static void exercise7() throws Exception {
        Path file = Files.createTempFile("c", ".txt");
        Files.writeString(file, "hello C", StandardCharsets.UTF_8);
        String s = Stream.of(file)
                .map(uncheck(Files::readString))
                .findFirst().orElse("");
        System.out.println("Ex7: " + s);
    }

    static void exercise8() {
        Stats stats = Stream.of(5, 3, 9, 1, 5, 7)
                .collect(Collector.of(
                        Stats::new,         // supplier
                        Stats::add,         // accumulator
                        Stats::combine      // combiner (for parallel)
                ));
        System.out.println("Ex8: " + stats);
    }

    static void exercise9() {
        // Bad pattern: shared mutable list + parallel forEach
        List<Integer> target = Collections.synchronizedList(new ArrayList<>());
        IntStream.range(0, 10_000).parallel().forEach(target::add);
        System.out.println("Ex9 (bad): size=" + target.size()); // may be < 10000 or inconsistent order

        // Good pattern: collect instead of mutating shared state
        List<Integer> safe = IntStream.range(0, 10_000).parallel().boxed().collect(Collectors.toList());
        System.out.println("Ex9 (good): size=" + safe.size());
    }
}
```

---

# 13) How to Run

```bash
# 1) Create folder and save files
mkdir -p lab && cd lab
# Save the code blocks as Main.java and Solutions.java in this folder

# 2) Compile and run solutions
javac *.java && java Solutions
```

Expected outputs (abbreviated):
```
Ex1: [ALICE, CHARLIE]
Ex2: [Alice Yellow(28), Bob Yellow(22), Alice Zephyr(30)]
Ex3: {a=2, b=2, c=2}
Ex4: sum=...
Ex5: Saved ID-42
Ex6:
hello A
hello B
Ex7: hello C
Ex8: Stats{count=6,sum=30,min=1,max=9,avg=5.00}
Ex9 (bad): size=<often < 10000>
Ex9 (good): size=10000
```

---

# 14) Further Practice (Optional)

- Re‑implement Exercise 8 using built‑in `IntSummaryStatistics`.
- Try `Collectors.teeing` (Java 12+) to compute min+max in one pass and merge.
- Refactor Exercise 5 to use `thenCombine` for parallel fetch of two resources.
- Add timeouts: `orTimeout`, `completeOnTimeout` (Java 9+).

---

## License
MIT — use freely in your GitHub repo.
