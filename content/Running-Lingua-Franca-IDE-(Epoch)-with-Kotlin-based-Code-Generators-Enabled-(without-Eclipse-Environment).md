You can use the Lingua Franca IDE (Epoch) for Kotlin-based code generators even without the Eclipse-based setup. You need Maven (mvn command) installed on your machine a priori.

For macOS X, inside the Lingua Franca repository (in this example, `lingua-franca-intellij`), run:
```
mvn package
```

This may fail with errors that look like due to cached Java code generated from Xtend, for example:

```
[ERROR] /Users/hokeunkim/Development/lingua-franca-intellij/org.lflang/xtend-gen/org/lflang/generator/CGenerator.java:[652]
[ERROR] 	final String cFilename = this.getTargetFileName(this.topLevelName);
[ERROR] 	                              ^^^^^^^^^^^^^^^^^
```

In this case, forcibly clean all cached data from the previous build with the following command:
```
./gradlew clean
```

If the `mvn package` command succeeded, now you can find the built Lingua Franca IDE in the target directory. For macOS X, run the following command to locate and run the Lingua Franca IDE IDE.

```
cd org.lflang.rca/target/products/org.lflang.rca/macosx/cocoa/x86_64/lflang.app/Contents/MacOS
./eclipse
```

You can use the graphical interfaces (diagrams) and the compilation with the Lingua Franca IDE for the Kotlin-based code generators, including C++ and TypeScript. For more information about using the Lingua Franca IDE (e.g., how to open Lingua Franca projects), see
[[Developer-Eclipse-Setup-with-Oomph#using-the-lingua-franca-ide]].