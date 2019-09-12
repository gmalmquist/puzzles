# NoDots Challenge
Java challenge: write "Hello world" without using any dots (`.`), backslashes (`\`), or semicolons (`;`).

A successful solution should look like:

```
$ ls
NoDots.java
$ grep '[.\\;]' NoDots.java
$ javac NoDots.java && java NoDots
Hello world
$
```

[Wouter's Challenge on Twitter](https://twitter.com/WouterCoekaerts/status/1171836094003974151)

Clarifying rules:
* The "Hello world" string must be printed by the output of `java NoDots`
  (not e.g. from a syntax error printed by the compiler).
* You have to actually be running java, no bash shenanigans `alias java NoDots='echo "Hello world"'`
* The only output of the program must be "Hello world", no surrounding error messages etc.
* You can't rely on anything clever being set in the environment, magic files existing
  outside of the NoDots.java file, or special flags being passed into the `java` command.
