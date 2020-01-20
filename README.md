# jar-classpath-bug
Demonstrates a problem with % character in Class-Path attribute of MANIFEST.MF.

It appears that Java doesn't support file paths that contain `%` in Class-Path attribute of MANIFEST.MF. I reproduced this problem on WIndows and on Linux, on  different versions of java up to 11.02.

## Set up
1. Class `Library` is in `lib.jar` which is copied into folders `bin/foo%3a` and `bin/foo_3a` (for comparison)
1. Class `Main` uses `Library` and has a main method. It's class file is in `bin`
1. `bin/bad-classpath.jar` is a classpath jar that refers to `foo%3a/lib.jar` in its MANIFEST.MF:
    ```
    Class-Path: foo%3a/lib.jar
    ```
1. `bin/good-classpath.jar` is a classpath jar that refers to `foo_3a/lib.jar` in its MANIFEST.MF:
    ```
    Class-Path: foo_3a/lib.jar
    ```

## Scenarios

### Without classpath jar:
Run `java -cp bin/foo%3a/lib.jar:bin Main`. This completes successfully and prints
```
Loaded library: Library@2c8d66b2
```

### With good-classpath.jar:
Run `java -cp bin/good-classpath.jar:bin Main`. This also works fine:
```
Loaded library: Library@35bbe5e8
```

### With bad-classpath.jar:
Run `java -cp bin/bad-classpath.jar:bin Main`. This time java can't find the `Library` class in `foo%3a/lib.jar`
```
Exception in thread "main" java.lang.NoClassDefFoundError: Library
        at Main.main(Main.java:3)
Caused by: java.lang.ClassNotFoundException: Library
        at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:582)
        at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:190)
        at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:499)
        ... 1 more
```
