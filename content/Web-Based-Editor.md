**FIXME:** This currently does not work. At one point it worked, however, so presumably it could be resurrected.

To create a web-based Lingua-Franca editor,

```
cd $LF/xtext
./gradlew jettyRun
```

If you then point your browser to http://localhost:8080, you should get a web-based editor. The above command will exit and shut down the web server when you hit return, so be sure to point your browser to localhost:8080 before typing return.

**FIXME:** What can I do with the web-based Lingua-Franca editor?  If I paste in the Minimal example from above, what happens?
