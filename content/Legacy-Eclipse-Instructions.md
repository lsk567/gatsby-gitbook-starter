The preferred way to set up Eclipse to develop Lingua Franca is using Oomph. See [[Developer Eclipse setup with Oomph]]. This page describes a more manual process. Be warned: Eclipse is a complex and brittle environment. Things can go wrong.

These instructions will set you up as an Eclipse-based developer of the Lingua Franca IDE and compiler, which is based on [xtext](https://www.eclipse.org/Xtext/documentation/index.html).

#### 1. Installation of Eclipse and Xtext

- [Install Eclipse](https://www.eclipse.org/downloads/)
- Start Eclipse in a new workspace (a *new* workspace is safest because Eclipse is brittle).
- Install Xtext in that workspace by following [these instructions](https://www.eclipse.org/Xtext/download.html).
- Install Kieler, which is used to render diagrams of reactor programs. Instructions are [here](https://github.com/icyphy/lingua-franca/wiki/diagrams#enabling-diagram-synthesis).

#### 2. Set up the Project

- In Eclipse, select File->Import->Team->Team Project Set
- Browse to `~/git/lingua-franca/xtext` (`$LF/xtext`) and select the file `LinguaFrancaProjectSetHTTPS.psf` (if you cloned using https) or `LinguaFrancaProjectSetSSH.psf` (if you cloned using ssh).  Note that even if the .psf files are located elsewhere, the .psf files create projects that refer to files in `~/git/lingua-franca/`.  If you accidentally cloned the `lingua-franca` repo anywhere other than in `~/git/`, then you could be confused about which files Eclipse is using.
- Close the Eclipse welcome page (which obscures the projects)
- You should have six projects (and a lot of build errors). Open the `org.icyphy.linguafranca` project.

#### 3. Build the Lingua Franca IDE

- In the Package Explorer, browse to src->org.icyphy->LinguaFranca.xtext
- Right click on that file and select Run As->Generate Xtext Artifacts. It will warn you that there are errors. Proceed.
- The project will rebuild, which may take some time. Hopefully, there are no build errors.

#### 4. Run the Lingua Franca IDE

To run the Lingua Franca IDE, right click on the first project in the PackageExplorer, org.icyphy.linguafranca, and select Run As->Eclipse Application. A new Eclipse opens.  This Eclipse is the Lingua Franca IDE. In this new Eclipse:
- Select File->New->Project  (a General Project is adequate).
- Give the project a name, like "test".
- **Uncheck Use default location** and specify a location that you can remember, such as `/Users/yourname/test`. The default is ridiculous.
- Close the Eclipse welcome window, if it is open. It obscures the project.
- Right click on the project name and select New->File.
- Give the new a name like "HelloWorld.lf" (with .lf extension).
- **IMPORTANT:** A dialog appears: Do you want to convert 'test' to an Xtext Project? Say YES.
- Start typing in Lingua-Franca! Try this:
```
  target C;
  main reactor HelloWorld {
      timer t;
      reaction(t) {=
          printf("Hello World.\n");
      =}
  }
```
When you save, generated code goes into your project directory, e.g. `/Users/yourname/test`.  That directory now has two directories inside it, `src-gen` and `bin`. The first contains the generated C code and the second contains the resulting executable program. Run the program:
```
    cd /Users/yourname/test
    bin/HelloWorld
```
This should produce output that looks something like this:
```
Start execution at time Sun Oct  6 08:42:49 2019
plus 314266000 nanoseconds.
Hello World.
Elapsed logical time (in nsec): 0
Elapsed physical time (in nsec): 1106000
```
  This should print "Hello World".


#### Working on the Lingua-Franca Compiler

The source code for the compiler is in the package `org.icyphy.linguafranca`.

- The grammar is in `src/org.icyphy/LinguaFranca.xtext`
- The code generator for the C target is in `src/org.icyphy.generator/CGenerator.xtend`
- The code generator for the TypeScript target is in `src/org.icyphy.generator/TypeScriptGenerator.xtend`
- The code generator for the C++ target is in `src/org.icyphy.generator/CppGenerator.xtend`
- To add a code generator for a new target, edit `src/org.icyphy.generator/CppGenerator.xtend`

#### Troubleshooting

Eclipse is astonishingly brittle and the setup is rather complex. Many things can go wrong. Here are some hints for getting things working:

- If you are a Mac user running macOS Catalina, you may need JDK 14 (or above) to launch the Lingua Franca IDE. More information about JDK 14 and Catalina's hardened runtime can be found [here](https://www.oracle.com/technetwork/java/javase/using-jdk-jre-macos-catalina-5781620.html).

- If you did not clone the repo into the recommended directory, then importing the Team Project Set from `LinguaFrancaProjectSetHTTPS.psf` will result in Eclipse cloning a fresh copy of the lingua-franca repository; the clone will be located in `~/git/lingua-franca` (it will not use the clone that you obtained the `LinguaFrancaProjectSet.psf` from!). Note that, if you already have a clone in `~/git/lingua-franca`, that will simply be reused (not overwritten).

- Use `git status` to check for modified files and untracked files. Particularly suspicious are files with names like `.classpath` and `.project`. It is mysterious how these get modified, but we recommend trying `git stash` and removing all untracked files that you did not create yourself.  Alternatively, you can start with a fresh clone of the repo.

- You may have files in `src-gen` directories that are interfering with the build processes. For example, if you run `npm` to install Node.js modules, you will likely get many files that the build system will then try to compile. In theory, you should be able to remove all src-gen directories and then start again by generating the Xtext artifacts, as recommended above.


