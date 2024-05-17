# DPE University Training

<p align="left">
<img width="10%" height="10%" src="https://user-images.githubusercontent.com/120980/174325546-8558160b-7f16-42cb-af0f-511849f22ebc.png">
</p>

## Gradle Incremental Builds and Local Build Caching Exercise

This is a hands-on exercise to go along with the
[Incremental Builds and Build Caching](https://dpeuniversity.gradle.com/app/catalog)
training module. In this exercise you will go over the following:

* Recap of Incremental Builds
* Enable and use Local Cache
* Order of lookup for cached outputs

---
## Prerequisites

* Finished going through the relevant sections in the training course
* JDK 1.8+ and recent version of Gradle build tool installed
    * https://gradle.org/install/
* Gradle Build Tool experience
    * Knowledge of core concepts
    * Authoring build files
    * Kotlin experience a plus but not required
* Basic experience with Java software development

---
## Recap of Incremental Cache and Limitations

1. Open the Gradle project in this repository in an editor of your choice
2. Inspect the build and source files, there is only one subproject called `app`
3. Run tests on the project. Notice the task outcome labels indicate
   that most tasks' actions were executed.

```diff
$ ./gradlew :app:test
- Task :app:compileJava
# Task :app:processResources NO-SOURCE
- Task :app:classes
- Task :app:compileTestJava
# Task :app:processTestResources NO-SOURCE
- Task :app:testClasses
- Task :app:test
```

4. Run the tests again. Notice that the outcome labels indicate that most
   tasks' actions were **not** executed. The output from the previous execution
   in the `build` directory was used.

```diff
$ ./gradlew :app:test
> Task :app:compileJava UP-TO-DATE
# Task :app:processResources NO-SOURCE
> Task :app:classes UP-TO-DATE
> Task :app:compileTestJava UP-TO-DATE
# Task :app:processTestResources NO-SOURCE
> Task :app:testClasses UP-TO-DATE
> Task :app:test UP-TO-DATE
```

5. Make a change to the `app/src/main/java/com/gradle/lab/App.java` file, add
   2 more exclamation marks to the `Hello World!` string:

```java
public String getGreeting() {
    return "Hello World!!!";
}
```

6. Run the tests again. Notice that the actions of `compileJava` and `test` were
   executed since changes were made to the inputs for those tasks.

```diff
$ ./gradlew :app:test
- Task :app:compileJava
# Task :app:processResources NO-SOURCE
- Task :app:classes
> Task :app:compileTestJava UP-TO-DATE
# Task :app:processTestResources NO-SOURCE
> Task :app:testClasses UP-TO-DATE
- Task :app:test
```

7. Revert the changes made to `app/src/main/java/com/gradle/lab/App.java` and
   run the tests again. Notice that the actions of `compileJava` and `test` were
   executed since the most recent build had different inputs for those tasks.

```diff
$ ./gradlew :app:test
- Task :app:compileJava
# Task :app:processResources NO-SOURCE
- Task :app:classes
> Task :app:compileTestJava UP-TO-DATE
# Task :app:processTestResources NO-SOURCE
> Task :app:testClasses UP-TO-DATE
- Task :app:test
```

---
## Enable and Use Local Cache

1. Edit the `gradle.properties` file and add `org.gradle.caching=true` to it. The
   contents of the file now look like:

```properties
org.gradle.console=verbose
org.gradle.caching=true
```

2. Run the clean task and then the tests, this will populate the cache.

```diff
$ ./gradlew :app:clean :app:test
# Task :app:clean
- Task :app:compileJava
# Task :app:processResources NO-SOURCE
- Task :app:classes
- Task :app:compileTestJava
# Task :app:processTestResources NO-SOURCE
- Task :app:testClasses
- Task :app:test
```

3. Change the string in `app/src/main/java/com/gradle/lab/App.java` to have 2
   extra exclamations again. Run the tests, notice that the task actions were
   executed. The inputs and outputs for these changes are not in the cache.

```diff
$ ./gradlew :app:test
- Task :app:compileJava
# Task :app:processResources NO-SOURCE
- Task :app:classes
> Task :app:compileTestJava UP-TO-DATE
# Task :app:processTestResources NO-SOURCE
> Task :app:testClasses UP-TO-DATE
- Task :app:test
```

4. Revert the changes to `app/src/main/java/com/gradle/lab/App.java` and run
   the tests again. Notice that some task actions were skipped and the outputs
   from the cache were used. It is important to understand that the outputs
   from the cache are put into the build directory and that's how the outputs are available to the tasks.

```diff
$ ./gradlew :app:test
! Task :app:compileJava FROM-CACHE
# Task :app:processResources NO-SOURCE
> Task :app:classes UP-TO-DATE
> Task :app:compileTestJava UP-TO-DATE
# Task :app:processTestResources NO-SOURCE
> Task :app:testClasses UP-TO-DATE
! Task :app:test FROM-CACHE
```

5. If you run the tests again, you will notice that the outcome label indicates
   the output from the previous execution in the `build` directory was used.
   This shows the order in which Gradle will look for the cached outputs,
   first in the build directory, then in the cache.

```diff
$ ./gradlew :app:test
> Task :app:compileJava UP-TO-DATE
# Task :app:processResources NO-SOURCE
> Task :app:classes UP-TO-DATE
> Task :app:compileTestJava UP-TO-DATE
# Task :app:processTestResources NO-SOURCE
> Task :app:testClasses UP-TO-DATE
> Task :app:test UP-TO-DATE
```

6. If you use the **-i** flag you can see the cache key used for each
   task input. Run a clean followed by the tests to see this.

```bash
$ ./gradlew :app:clean :app:test -i
...
...
...
> Task :app:test FROM-CACHE
Build cache key for task ':app:test' is 5eea5b00209094c7c5d641d592af0a2b
Task ':app:test' is not up-to-date because:
  Output property 'binaryResultsDirectory' file /Users/adayal/trainings/dpeuni-gradle-incremental-and-caching-local/app/build/test-results/test/binary has been removed.
  Output property 'binaryResultsDirectory' file /Users/adayal/trainings/dpeuni-gradle-incremental-and-caching-local/app/build/test-results/test/binary/output.bin has been removed.
  Output property 'binaryResultsDirectory' file /Users/adayal/trainings/dpeuni-gradle-incremental-and-caching-local/app/build/test-results/test/binary/output.bin.idx has been removed.
  Output property 'binaryResultsDirectory' file /Users/adayal/trainings/dpeuni-gradle-incremental-and-caching-local/app/build/test-results/test/binary/results.bin has been removed.
  Output property 'reports.enabledReports.html.outputLocation' file /Users/adayal/trainings/dpeuni-gradle-incremental-and-caching-local/app/build/reports/tests/test has been removed.
Loaded cache entry for task ':app:test' with cache key 5eea5b00209094c7c5d641d592af0a2b
```

7. You can also use the **-Dorg.gradle.caching.debug=true** flag to view more details.

```bash
$ ./gradlew :app:clean :app:test -Dorg.gradle.caching.debug=true
```

8. You can see the cache entries by looking in the cache directory:

```bash
$ ls -ltr ~/.gradle/caches/build-cache-1/
```

---
## Local Cache Configuration

1. By default items will expire from the local cache after 7 days. Let's
   modify the configuration to keep items for 30 days. Edit the `~/.gradle/init.gradle.kts` file and put the following configuration:

```kotlin
gradle.settingsEvaluated {
     buildCache {
        local {
            removeUnusedEntriesAfterDays = 30
        }
    }
}
```

2. Run a clean then the tests to make sure there was no mistake in the configuration:

```diff
$ ./gradlew :app:clean :app:test
# Task :app:clean
! Task :app:compileJava FROM-CACHE
# Task :app:processResources NO-SOURCE
> Task :app:classes UP-TO-DATE
! Task :app:compileTestJava FROM-CACHE
# Task :app:processTestResources NO-SOURCE
> Task :app:testClasses UP-TO-DATE
! Task :app:test FROM-CACHE
```

---
## Solution Reference

If you get stuck you can refer to the `solution` branch of this repository.