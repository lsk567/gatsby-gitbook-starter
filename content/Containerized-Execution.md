For the `C` target at least, the Lingua Franca code generator is able to generate a Dockerfile when it generates the C source files. To enable this, include the `docker` property in your target specification, as follows:
```
target C {
    docker: true
};
```
The generated Docker file has the same name as the LF file except that the extension is `.Dockerfile` and will be put in the `src-gen` directory.  You can also specify options.  Currently, only the base image (`FROM`) can be customized, but this will be extended to allow further customization is the future.  To customize the Docker file, instead of just `true` above, which gives default options, specify the options as in the following example:
```
target C {
    docker: {FROM: "alpine:latest"}
};
```
This specifies that the base image is the latest version of `alpine`, a very small Linux. In fact, `alpine:latest` is the default value for this option, so you only need to specify this option if you need something other than `alpine:latest`.

How to use this depends on whether your application is federated. You will need to [install Docker](https://docs.docker.com/get-docker/) if you haven't already in order to use this.

## Unfederated Execution

Suppose your LF source file is `Foo.lf`.  When you run `lfc` or use the IDE to generate code, a file called `Foo.Dockerfile` will appear in the `src_gen` directory.  You can use this file to build a Docker image as follows. First, make sure you are in the same directory as the source file. Then issue the command:
```
   docker build -t foo -f src-gen/Foo.Dockerfile . 
```
This will create a Docker image with tag `foo`. The tag is required to be all lower-case letters. By convention, we advise using the LF source file name, converted to lower case.

You can then use this tag to run the image in a container:
```
    docker run -t --rm foo
```
The `-t` option creates a pseudo terminal, which is necessary for you to see any output produced by your program to `stdout`.  If your program also reads from `stdin`, then you will need to give the `-i` option as well, or combine the two as `it`.

The `--rm` option is important. This removes the container upon completion of the run. If you omit this option, the container will continue to exist even after your program has terminated. You can alternatively remove the container after the run using `docker rm`.

If you wish for your program to run in the background, give a `-d` option as well (for "detached"). In this case, you will not see any output from your run.

The above run command can include any supported command-line arguments to the LF program. For example, to specify a logical timeout, you can do this:
```
    docker run -t --rm foo --timeout 20 sec
```

## Federated Execution

For a federated Lingua Franca program, one Dockerfile is created for each federate plus an additional one for the RTI.  You will need to run `docker build` for each of these. For example, to build the image for the RTI, you can do this:
```
docker build -t distributedcount_rti -f src-gen/DistributedCount_RTI.Dockerfile .
```
(This is for the [DistributedCount.lf](https://github.com/icyphy/lingua-franca/blob/master/test/C/DistributedCount.lf) test program).

Now, there are several options for how to proceed.  One is to create a named network on which to run your federation. For example, to create a network named `lf`, do this:
```
    docker network create lf
```
You can then run the RTI on this network and assign the RTI a name that the federates can use to find the RTI:
```
    docker run -t --rm --network lf --name distributedcount-rti distributedcount_rti
```
Here, the assigned name is not quite the same as the tag that was specified when building the image (the last argument is the tag of the image to run in a container) because a host name is not allowed to have an underscore in it.

Currently, you will also have to specify this host name in the LF source file so that the federates know where to find the RTI. E.g., in [DistributedCount.lf](https://github.com/icyphy/lingua-franca/blob/master/test/C/DistributedCount.lf), change the end of the file to read as follows:
```
federated reactor DistributedCount at distributedcount-rti {
    c = new Count();
    p = new Print();
    c.out -> p.in after 200 msec;
}
```
Notice the `at distributedcount-rti`, which must match the name you use when running the RTI. **FIXME:** We should find a way to make this more automatic!

In two other terminals, you can now run the other federates on the same network:
```
docker run -t --rm --network lf distributedcount_c
```
and
```
docker run -t --rm --network lf distributedcount_p
```
