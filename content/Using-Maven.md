The Lingua Franca infrastructure can be compiled and the tests can be run using Maven (a build system for Java-based projects). We do not recommend this method, however, because it is astonishingly slow. To compile LF and run the regression tests:

```
cd $LF/xtext
mvn compile
mvn verify
```

To run the tests a bit faster, you can do this:

```
mvn -T 1C -o test -pl :org.icyphy.linguafranca.tests -am
```

To run the tests in just one file:

```
mvn -Dtest=LinguaFrancaGeneratorTest test
```

For more details, see [[Maven Notes]].