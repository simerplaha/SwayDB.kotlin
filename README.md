# SwayDB.kotlin.examples [![Slack Chat][slack-badge]][slack-link] [![Gitter Chat][gitter-badge]][gitter-link] [![Build status][build-badge]][build-link] [![Maven central][maven-badge]][maven-link]

[gitter-badge]: https://badges.gitter.im/Join%20Chat.svg
[gitter-link]: https://gitter.im/SwayDB-chat/Lobby

[slack-badge]: https://img.shields.io/badge/slack-join%20chat-e01563.svg
[slack-link]: https://join.slack.com/t/swaydb/shared_invite/enQtNzI1NzM1NTA0NzQxLTJiNjRhMDg2NGQ3YzBkNGMxZGRmODlkN2M3MWEwM2U2NWY1ZmU5OWEyYTgyN2ZhYjlhNjdlZTM3YWJjMGZmNzQ

[maven-badge]: https://img.shields.io/maven-central/v/io.swaydb/java_2.12.svg
[maven-link]: https://search.maven.org/search?q=g:io.swaydb%20AND%20a:java_2.12

[build-badge]: https://travis-ci.com/simerplaha/SwayDB.kotlin.examples.svg?branch=master
[build-link]: https://travis-ci.com/simerplaha/SwayDB.kotlin.examples

Implements examples demoing [SwayDB](https://github.com/simerplaha/SwayDB)'s Java API.

Requirements
============

Kotlin 1.3 and later.

### Quick start example.

See [QuickStart.kt](/src/main/kotlin/quickstart/QuickStart.kt).


```kotlin
val map =
  MapConfig
    .withFunctions(intSerializer(), intSerializer())
    .init();

map.put(1, 1) //basic put
map.get(1).get() //basic get
map.expire(1, Duration.ofSeconds(1)) //basic expire
map.remove(1) //basic remove

//atomic write a Stream of key-value
map.put(Stream.range(1, 100).map { item -> KeyVal.create(item) })

//create a read stream from 10th key-value to 90th, increment values by 1000000 and insert.
val updatedKeyValuesStream =
  map
    .from(10)
    .stream()
    .takeWhile { keyVal -> keyVal.key() <= 90 }
    .map { keyVal -> KeyVal.create(keyVal.key(), keyVal.value() + 5000000) }

map.put(updatedKeyValuesStream)

val function = PureFunction.OnKeyValue<Int, Int, Return.Map<Int>> { key, value, _ ->
  when {
    key < 25 -> //remove if key is less than 25
      Return.remove()
    key < 50 -> //expire after 2 seconds if key is less than 50
      Return.expire(Duration.ofSeconds(2))
    key < 75 -> //update if key is < 75.
      Return.update(value!! + 10000000)
    else -> //else do nothing
      Return.nothing()
  }
}

map.registerFunction(function) //register the function.

map.applyFunction(1, 100, function) //apply the function to all key-values ranging 1 to 100.

//print all key-values to view the update.
map
  .stream()
  .forEach { println(it) }
```
