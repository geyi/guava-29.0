# Bloom Filter
因为Redisson的普通版本不支持在集群模式下使用布隆过滤器，所以我基于Guava中提供的布隆过滤器算法，修改了对比特索引位进行操作的get及set方法，
将Guava中基于Java的AtomicLongArray的本地存储方式修改为基于Redis的bitmaps的存储方式。

Guava源码的修改很简单，如下：
```diff
diff --git a/guava/pom.xml b/guava/pom.xml
index 7483f57..7df60a0 100644
--- a/guava/pom.xml
+++ b/guava/pom.xml
@@ -8,6 +8,7 @@
     <version>29.0-jre</version>
   </parent>
   <artifactId>guava</artifactId>
+  <version>29.1-jre</version>
   <packaging>bundle</packaging>
   <name>Guava: Google Core Libraries for Java</name>
   <description>
diff --git a/guava/src/com/google/common/hash/HashCode.java b/guava/src/com/google/common/hash/HashCode.java
index b6a5ff8..6f47029 100644
--- a/guava/src/com/google/common/hash/HashCode.java
+++ b/guava/src/com/google/common/hash/HashCode.java
@@ -97,7 +97,7 @@ public abstract class HashCode {
    * byte-based hashcode. Otherwise it returns {@link HashCode#asBytes}. Do <i>not</i> mutate this
    * array or else you will break the immutability contract of {@code HashCode}.
    */
-  byte[] getBytesInternal() {
+  public byte[] getBytesInternal() {
     return asBytes();
   }
 
@@ -302,7 +302,7 @@ public abstract class HashCode {
     }
 
     @Override
-    byte[] getBytesInternal() {
+    public byte[] getBytesInternal() {
       return bytes;
     }
 
diff --git a/guava/src/com/google/common/hash/LongAddable.java b/guava/src/com/google/common/hash/LongAddable.java
index a95eece..a034d2f 100644
--- a/guava/src/com/google/common/hash/LongAddable.java
+++ b/guava/src/com/google/common/hash/LongAddable.java
@@ -19,7 +19,7 @@ package com.google.common.hash;
  *
  * @author Louis Wasserman
  */
-interface LongAddable {
+public interface LongAddable {
   void increment();
 
   void add(long x);
diff --git a/guava/src/com/google/common/hash/LongAddables.java b/guava/src/com/google/common/hash/LongAddables.java
index d2768bc..e5d2ad4 100644
--- a/guava/src/com/google/common/hash/LongAddables.java
+++ b/guava/src/com/google/common/hash/LongAddables.java
@@ -22,7 +22,7 @@ import java.util.concurrent.atomic.AtomicLong;
  *
  * @author Louis Wasserman
  */
-final class LongAddables {
+public final class LongAddables {
   private static final Supplier<LongAddable> SUPPLIER;
 
   static {

```

然后我在我的另一个项目[spring-cloud](https://github.com/geyi/spring-cloud/tree/master/base/src/main/java/com/airing/spring/cloud/base/utils/bloom)
中基于以上实现了一个简单的布隆过滤器。

# Guava: Google Core Libraries for Java

[![Latest release](https://img.shields.io/github/release/google/guava.svg)](https://github.com/google/guava/releases/latest)
[![Build Status](https://travis-ci.org/google/guava.svg?branch=master)](https://travis-ci.org/google/guava)

Guava is a set of core Java libraries from Google that includes new collection types
(such as multimap and multiset), immutable collections, a graph library, and
utilities for concurrency, I/O, hashing, caching, primitives, strings, and more! It
is widely used on most Java projects within Google, and widely used by many
other companies as well.

Guava comes in two flavors.

*   The JRE flavor requires JDK 1.8 or higher.
*   If you need support for JDK 1.7 or Android, use the Android flavor. You can
    find the Android Guava source in the [`android` directory].

[`android` directory]: https://github.com/google/guava/tree/master/android

## Adding Guava to your build

Guava's Maven group ID is `com.google.guava` and its artifact ID is `guava`.
Guava provides two different "flavors": one for use on a (Java 8+) JRE and one
for use on Android or Java 7 or by any library that wants to be compatible with
either of those. These flavors are specified in the Maven version field as
either `29.0-jre` or `29.0-android`. For more about depending on Guava, see
[using Guava in your build].

To add a dependency on Guava using Maven, use the following:

```xml
<dependency>
  <groupId>com.google.guava</groupId>
  <artifactId>guava</artifactId>
  <version>29.0-jre</version>
  <!-- or, for Android: -->
  <version>29.0-android</version>
</dependency>
```

To add a dependency using Gradle:

```gradle
dependencies {
  // Pick one:

  // 1. Use Guava in your implementation only:
  implementation("com.google.guava:guava:29.0-jre")

  // 2. Use Guava types in your public API:
  api("com.google.guava:guava:29.0-jre")

  // 3. Android - Use Guava in your implementation only:
  implementation("com.google.guava:guava:29.0-android")

  // 4. Android - Use Guava types in your public API:
  api("com.google.guava:guava:29.0-android")
}
```

For more information on when to use `api` and when to use `implementation`,
consult the
[Gradle documentation on API and implementation separation](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_separation).

## Snapshots and Documentation

Snapshots of Guava built from the `master` branch are available through Maven
using version `HEAD-jre-SNAPSHOT`, or `HEAD-android-SNAPSHOT` for the Android
flavor.

-   Snapshot API Docs: [guava][guava-snapshot-api-docs]
-   Snapshot API Diffs: [guava][guava-snapshot-api-diffs]

## Learn about Guava

-   Our users' guide, [Guava Explained]
-   [A nice collection](http://www.tfnico.com/presentations/google-guava) of
    other helpful links

## Links

-   [GitHub project](https://github.com/google/guava)
-   [Issue tracker: Report a defect or feature request](https://github.com/google/guava/issues/new)
-   [StackOverflow: Ask "how-to" and "why-didn't-it-work" questions](https://stackoverflow.com/questions/ask?tags=guava+java)
-   [guava-announce: Announcements of releases and upcoming significant changes](http://groups.google.com/group/guava-announce)
-   [guava-discuss: For open-ended questions and discussion](http://groups.google.com/group/guava-discuss)

## IMPORTANT WARNINGS

1.  APIs marked with the `@Beta` annotation at the class or method level are
    subject to change. They can be modified in any way, or even removed, at any
    time. If your code is a library itself (i.e. it is used on the CLASSPATH of
    users outside your own control), you should not use beta APIs, unless you
    [repackage] them. **If your code is a library, we strongly recommend using
    the [Guava Beta Checker] to ensure that you do not use any `@Beta` APIs!**

2.  APIs without `@Beta` will remain binary-compatible for the indefinite
    future. (Previously, we sometimes removed such APIs after a deprecation
    period. The last release to remove non-`@Beta` APIs was Guava 21.0.) Even
    `@Deprecated` APIs will remain (again, unless they are `@Beta`). We have no
    plans to start removing things again, but officially, we're leaving our
    options open in case of surprises (like, say, a serious security problem).

3.  Guava has one dependency that is needed at runtime:
    `com.google.guava:failureaccess:1.0.1`

4.  Serialized forms of ALL objects are subject to change unless noted
    otherwise. Do not persist these and assume they can be read by a future
    version of the library.

5.  Our classes are not designed to protect against a malicious caller. You
    should not use them for communication between trusted and untrusted code.

6.  For the mainline flavor, we unit-test the libraries using only OpenJDK 1.8
    on Linux. Some features, especially in `com.google.common.io`, may not work
    correctly in other environments. For the Android flavor, our unit tests run
    on API level 15 (Ice Cream Sandwich).

[guava-snapshot-api-docs]: https://guava.dev/releases/snapshot-jre/api/docs/
[guava-snapshot-api-diffs]: https://guava.dev/releases/snapshot-jre/api/diffs/
[Guava Explained]: https://github.com/google/guava/wiki/Home
[Guava Beta Checker]: https://github.com/google/guava-beta-checker

<!-- References -->

[using Guava in your build]: https://github.com/google/guava/wiki/UseGuavaInYourBuild
[repackage]: https://github.com/google/guava/wiki/UseGuavaInYourBuild#what-if-i-want-to-use-beta-apis-from-a-library-that-people-use-as-a-dependency

