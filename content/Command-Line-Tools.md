Here we assume that you have either [installed the command-line tools](Downloading-and-Building#command-line-tools), [cloned the repository](Downloading-and-Building#clone-the-repository), or [set up the developer's Eclipse](Developer-Eclipse-Setup-with-Oomph). In the following, we assume that `$LF` is the root directory of your Lingua Franca installation (for me, having set up the developer's Eclipse, this is `/Users/eal/lingua-franca-master/git/lingua-franca`). In this directory is a subdirectory $LF/bin. This contains the following files:
* `lfc`: A command-line compiler.
* `build-lfc`: Script to build the compiler (`lfc` uses this to build itself if necessary).
* `run-lf-tests`: A command to run all the Lingua Franca tests (see [[Regression Tests]]).

**It is recommended to add `$LF/bin` to the `PATH` environment variable.**
On the rest of this page, we assume that `$LF/bin` is on the `PATH`.

### `lfc`

You can invoke the compiler on a Lingua Franca file as follows:
```
    lfc filename.lf
```
This will generate subdirectories `src-gen` and `bin` containing the generated and compiled code, respectively.  You can give a `-r` or `--rebuild` option to `lfc`, in which case it will rebuild the compiler (using the offline mode) before invoking it.

```
    Usage: lfc [options] file... [target-options]
    Options:
      -c | --target-compiler  Target compiler to invoke.
      -h | --help             Display this information.
      -n | --no-compile       Do not invoke target compiler.
      -r | --rebuild          Rebuild the compiler first.
```

To use the compiler, first create a directory in which you would like to work and create a file in that directory called `HelloWorld.lf` containing the following:
```
  target C;
  main reactor HelloWorld {
      timer t;
      reaction(t) {=
          printf("Hello World.\n");
      =}
  }
```
Then, in that directory, invoke the compiler:
```
    lfc HelloWorld.lf
```
This will create two directories inside your working directory, `src-gen` and `bin`. The first contains the generated C code and the second contains the resulting executable program. Run the program:
```
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

### `build-lfc`
Running `build-lfc` will build the Lingua Franca compiler using Gradle. The command-line options are:
```
    -c | --clean:          Build entirely from scratch.
    -h | --help:           Display this information.
    -o | --offline:        Use cached libraries.
    -s | --stacktrace:     Provide stacktrace of build errors.
```
The first time you run this, you will need a network connection because a certain amount of infrastructure gets downloaded. After that, you can specify the `--offline` option to build using local files only.

### `run-lf-tests`

See [[Regression Tests]].