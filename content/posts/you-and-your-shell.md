+++
title = "You and Your Shell"
date = 2023-09-24
description = "What is a shell? Baby don't glob me."

draft = true
+++

This article is my attempt to explain what is a shell and what it does. My goal is for you to build a useful mental model of what's going on under the hood when you open a terminal, paste some command, and hope that it doesn't erase all your terrible, embarrasing, hidden sci-fi poems.

# Is this for you?

I think you won't get much from this article if you can answer these questions without even blinking:

- What is the difference between `echo foo` and `echo 'foo'`?
- What's going on when you do `DEBUG='*' some-command`
- What's the difference between a program and a shell built-in? Why do we have both?

But you may still want to jump directly to [the examples](#examples). If you find any of them surprising, then maybe there's something useful for you here!

# What is a shell?

My favorite definition comes from the classic [The UNIX Programming Environment](https://en.wikipedia.org/wiki/The_Unix_Programming_Environment):

> [The shell is] the program that interprets your requests to run programs.

This short sentence tells us a lot:

- First, a shell is just a program. `bash`, `zsh` and `fish` are all different programs that function as shells.
- Second, a shell interprets requests. You use a specific language for these requests, even if you are not aware of it.
- Finally, a shell is used to run programs. You can do other things with it, but that's its main purpose.

Let's explore this definition in more detail.

## Running programs

The first thing we need to understand is that a program is always started from another program.[^1] And who started that other program, you might ask? Well, of course: a third program. And who started that one? Yet another program. But who...? I don't know, [a turtle or something](https://en.wikipedia.org/wiki/Init).

The details of what exactly a program does to start another one are not relevant for our purposes. We can just assume that there is a function that looks like this:

```typescript
function runProgram(
  pathToProgram: string,
  arguments: string[],
  environmentVariables?: Record<string, string>
)
```

This is a crude simplification, but it will do. At its core, starting a program requires its path in the filesystem and a list of arguments. We can also optionally pass a dictionary of environment variables.

We can use this function and our previous definition to write a very simple and useless shell.

## The worst shell in the world

Since we defined a shell as a program that interprets requests to run programs and assumed we have a `runProgram` function, we can write an extremely simple shell that just runs whatever program is passed to it:

```typescript
while (true) {
  const pathToProgram = readLine();
  runProgram(pathToProgram, []);
}
```

This would be extremely useless, but it does the job. You input a path to the program you want to run, our shitty shell interprets your request verbatim, and that's it. But a real shell does much more than that.

## Finding programs

When you want to run the `df` program to see if you have enough space to keep ~~downloading~~ buying movies, you don't write `/usr/bin/df`, you just write `df`. This wouldn't work on our dumb implementation, but it does in any real shell.

The reason this works is that the shell will check a list of pre-defined directories to see if any of them contains a program with that name. You probably already know that this list is in the [`PATH` environment variable](https://en.wikipedia.org/wiki/PATH_(variable)). We can easily add this feature to our previous snippet, especially if we assume that we have more magic functions available:

```typescript
while (true) {
  const program = readLine();
  const pathToProgram = findProgram(process.env.PATH, program);

  if (pathToProgram) {
      runProgram(pathToProgram, []);
  } else {
      console.error(`No program named '${program}' was found`);
  }
}
```

_Now_ we are doing some interpretation.

But what if we want to pass some arguments to a program? If we run `rm Titanic.mkv` we'll get this:

```
No program named 'rm Titanic.mkv' was found
```

Which makes sense! We are taking the full line and interpreting it as a program name. Whitespace is not special, and neither are you.

What we need here is some more of that sweet _interpretation_.

## Parsing arguments

Let's update our shell to interpret whitespace as a separator betwen the name of the program and the arguments:

```typescript
while (true) {
  const line = readLine();

  const [program, ...args] = line.split(/\s+/g);

  const pathToProgram = findProgram(process.env.PATH, program);
  if (pathToProgram) {
      runProgram(pathToProgram, args);
  } else {
      console.error(`No program named '${program}' was found`);
  }
}
```

If this program actually worked and we ran `rm Titanic.mkv` on it, it would actually delete the file. Probably. In any case, the takeaway is that we are doing a more complex interpretation of the command we receive. The command is still just a string, but we are starting to parse it in a somewhat interesting way.

## Globs

What if you want to remove all the files that end in `.mkv` in a directory? It's easy with our shell: you just run `ls`, then write `rm `, and then manually add all the files that end with that extension. I'm sure you won't make any typos.

If you don't like typing, then there are some ways this could be automated. Maybe you could use a version of `rm` that, instead of a list of files, receives an extension as its argument and removes all the files with that extension. Or perhaps you could have one program that writes all the files with some extension to a file, and then pass that file to another program which consumes it and removes all the files pointed by it...

...or you could add a new feature to the shell. You could say that, if the shell finds something like `*.mkv`, then it will interpret it as "all the files in the current directory that end in mkv". This is called [glob expansion or globbing](https://tldp.org/LDP/abs/html/globbingref.html). [Globs](https://en.wikipedia.org/wiki/Glob_(programming)) are like regular expressions, but worse, or better; it's hard to say.

The important part here, and probably the most important part of this article, is that **this expansion is performed by the shell!** This is quite easy to check by writing a program that just prints its arguments. With Node.js, you can do something like this:

```typescript
// index.js
const arguments = process.argv.slice(2);
console.log(JSON.stringify(arguments));
```

If you run `node index.js 1 2 foo`, you will get the list `["1","2","foo"]`. (Don't pay too much attention to the `.slice(2)` part, this is just to avoid printing the path to the node binary and the path to the script, which are included in the arguments list for what I'm sure are very good reasons.)

If you now create some files:

```
$ touch a.txt b.txt c.txt
```

And then run `node index.js *.txt`, the program will print `["a.txt","b.txt","c.txt"]`, not `["*.txt"]`.

Things like `*` are called **metacharacters**. These are characters that receive a special treatment by the shell.

But now you, being a smart and curious person, are wondering: "what if I actually want to pass `*.txt` as an argument to a program, for my own devious purposes?".

## Quotes

This is one of the things you might do without fully understanding. You run something in the shell; it doesn't work and you get a scary error message. You try adding some quotes. Now it works. Success! But what did actually happen?

Here I'm going to quote from *The UNIX Programming Environment* again.

> Given the number of shell metacharacters, there has to be some way to say to the shell, "Leave it alone". The easiest and best way to protect special characters from being interpreted is to enclose them in single quote characters:
>
> ```
> $ echo '***'
> ***
> $
> ```

Honestly, you should just close this tab and go read Chapter 3 from that book. It's so good.

But yes: you use single or double quotes to escape metacharacters. Single quotes escape everything, double quotes allows certain expansions within them. I think a good thumb of rule is "always use single quotes unless you know what you're doing". <small>It's also good for your hands because you can type single quotes without pressing shift.</small> <small style="font-size: x-small;">There is a pun here with "rule of thumb" and "fingers" but I couldn't think of anything.</small>

So, going back to the question in the previous section, if you want to literally pass `*.txt` as an argument to a program, you can do:

```
$ rm '*.txt'
```

And if you have a file that is literally named `*.txt` then 1) you are a monster and 2) it will be removed.

You can also try this with the little script we wrote before:

```
$ node index.js '*.txt'
["*.txt"]
```

## Environment variables

So far, we've called our `runProgram` function without passing any environment variables. The default behavior in this case is that the environment of the parent process will be passed to the child process (or maybe some of them are changed? I'm fuzzy on the details here).

What if you want to run a program with some specific environment variables? Most shells let you do this with a special syntax. In bash/zsh/fish, for example, you can do:

```
$ FOO=1 BAR=2 program arg1 arg2
```

That is, you can prepend the program name with a list of environment variables. As before, it's easy to write a little script to test this out:

```typescript
// index.js
console.log(process.env.FOO);
```

Running `node index.js` will print `undefined` but running `FOO=1 node index.js` will print `1`.

There is an alternative way to do this that you've surely seen before, but which is not exactly equivalent:

```
$ export FOO=1
$ node index.js
1
```

The difference here is that `export FOO=1` is modifying the environment of the shell, which is then passed down to our script. This is persistent: if we run `node index.js` a second time, it will print 1 again. But if we do that with the previous version (assuming a fresh shell instance), it won't:

```
$ FOO=1 node index.js
1
$ node index.js
undefined
```

## Built-in commands

The most interesting part of the previous section is `export FOO=1`. This is an example of running a command in the shell that _doesn't_ execute a program. `export` here is what's called a *built-in command* of the shell.

When you run things like `df`, `rm`, `vim` or whatever, there is an actual file in your disk with the code for that program. You can use `whereis` to find the path to that file:

```
$ whereis df
df: /usr/bin/df /usr/share/man/man1/df.1.gz
```

But if you do the same with a built-in, you get a sad void:

```
$ whereis export
export:
```

Why do we have built-ins? Why isn't everything a program?

In many cases, the reason is that the built-in does something that can't be done by a program. For example, `export` modifies the environment of the shell and `cd` changes its current directory. This is something that can't be done by a program, because a program can't make these modifications to its parent process.

There are also many built-in commands that have a program equivalent, which muddies the waters a bit. For example, `echo` is a built-in command in bash, but there is also a `/usr/bin/echo` program. I'm not completely sure why this is the case, but the answer seems to be "POSIX", a topic I won't delve into, but that will hopefully be easier to understand after reading this.

In any case, if you want to know if a command is a built-in or not, you can use the `type` command:

```
$ type export
export is a shell builtin
```

Type is, of course, a builtin:

```
$ type type
type is a shell builtin
```

# Examples!

Ok, now for the fun part. Let's use what we learned to explore some interesting examples.

## Example 1: Escaped globs

Which metacharacters are available depends on the shell you are using.[^2] Bash and zsh, for example, support the `**` metacharacter, which can be used to match all files under the current directory, even traversing directories. So if you have a directory like this:

```
$ tree
.
├── dir1
│   ├── file11.txt
│   └── file12.txt
├── dir2
│   ├── dir21
│   │   └── file211.txt
│   └── file21.txt
├── file1.txt
└── file2.txt
```

Then `echo *` and `echo **/*.txt` will produce different outputs:

```
$ echo *.txt
file1.txt file2.txt
$ echo **/*.txt
dir1/file11.txt dir1/file12.txt dir2/dir21/file211.txt dir2/file21.txt file1.txt file2.txt
```

This is a very common pattern to, for example, run all the tests under some directory. In the Node.js ecosystem it's normal to see an npm script like this:

```
"test": "mocha test/**/*.js"
```

But npm scripts, by default, are run using the `/bin/sh` shell, which is a link to an actual shell and whose target depends on the operating system. In Ubuntu, this corresponds to the [Dash shell](https://en.wikipedia.org/wiki/Almquist_shell#Dash), which doesn't support that kind of globbing. In this case, this means that Mocha will receive a string with `**/*.js` as its parameter.

So, why does this work? In this instance, the reason is that the Mocha test runner accepts globs as a parameter. So doing `mocha test/**/*.js` and `mocha 'test/**/*.js'` are somewhat equivalent.

Somewhat. What guarantees do we have that the glob expansion done by the shell is the same as the one done by Mocha? Absolutely none. So I think it's important to be aware of the difference here.

The same example with another test runner, Jest, is even more interesting because Jest accepts regexes as parameters, not globs, and `test/**/*.js` is not a valid regex, so you get a funny error.

## Example 2: Escaping quotes

If you want to pass an argument that actually contains a quote, you can escape it with the other type of quote. That is:

```
$ echo 'The "quick" brown fox'
The "quick" brown fox
$ echo "The 'quick' brown fox"
The 'quick' brown fox
```

If you want to include both kinds of quotes, you could do this:

```
$ echo "The 'quick' brown \"fox\""
```

But, as mentioned before, it's better to use single quotes by default. The problem is that you can't escape single quotes with a backslash in bash and zsh (you can in fish, but let's ignore that). So, if you want to pass a parameter that contains both double quotes and single quotes _and_ you want to use single quotes for most of it, you have to do this:

```
$ echo 'The '"'"'quick'"'"' brown "fox"'
The 'quick' brown "fox"
```

The key to understand this monstrosity is that you can concatenate quotes strings. All of these are equivalent:

```
$ echo foobarbaz
foobarbaz
$ echo "foo""bar""baz"
foobarbaz
$ echo 'foo'"bar"'baz'
foobarbaz
```

If that makes sense, and if you squint hard enough, then you can understand the previous example. Here it is again, which each part in a different line:

```
$ echo 'The '\
> "'"\
> 'quick'\
> "'"\
> ' brown "fox"'
The 'quick' brown "fox"
```

For a good, alternative explanation of this, check [this answer](https://stackoverflow.com/a/1250279/3055448) on StackOverflow.

## Example 3: Conditions in bash programming

So far everything we've seen is related to running programs. But the language interpreted by shells like bash is pretty much a full programming language, including things like conditionals. For example, you can do:

```bash
if [ 3 -gt 1 ]
then
  echo "3 is greater than 1"
else
  echo "Math is broken"
fi
```

Now, you might think that the syntax for an `if` statement is something like this:

```
if [ <conditions> ]
then
  <true>
else
  <false>
fi
```

Where `<conditions>` uses a terrible syntax. But the actual syntax is this:

```
if <command>
then
  <true>
else
  <false>
fi
```

Where `<true>` will be executed if the command exits with a zero status code, and `<false>` will be executd if the command exits with a non-zero code.

Wait, does that mean that...?

```
$ whereis [
[: /usr/bin/[ /usr/share/man/man1/[.1.gz
```

Yes: `[` is a program.[^3] It receives a list of arguments that are interpreted as conditions, and exits with a zero exit code if its arguments evaluate to true, and non-zero otherwise. Actually, `[` is kind of an alias to the `test` command.

```
$ test 3 -gt 1
$ echo $?
0
$ test 3 -gt 3
$ echo $?
1
```

(Here we are using the `$?` variable, which contains the exit code of the last command, to check the result of the `test` command.)

And I said "kind of an alias" because `[` is not exactly the same as `test`.

```
$ [ 3 -gt 1
bash: [: missing `]'
```

As you can check by running `man test`, `[` is pretty much the same as `test`, except that it requires a `]` as its last argument.

Fun stuff.

# Clarifications

There are a lot of simplifications in this article. Some of them are there to make the explanation easier to understand, and some of them are due to my own ignorance. This is a list of the ones I'm aware of and that I think are important:

- I didn't mention system calls at all, even if they are relevant for a more common definition of a shell: "a shell is a program that provides an interface to the operating system". This is probably a technically better definition, but I wanted to focus on the "interprets requests to run programs" part.
- The environment is actually a list of strings, not a dictionary. These strings, by convention, look like `key=value`.

# Further reading

[The UNIX Programming Environment](https://en.wikipedia.org/wiki/The_Unix_Programming_Environment) is a classic. It's a bit dated, but it's a great read anyway.

[Advanded Programming in the UNIX Environment](https://en.wikipedia.org/wiki/Advanced_Programming_in_the_Unix_Environment) is a more modern and technical book. I haven't read it cover to cover, not even 10% of that, but it's great as a reference.

# Footnotes

[^1]: I'm going to use "program" here for both "programs" and "processes". There are a million explanations out there about the difference, so I won't bother here.

[^2]: And also on the options you have enabled. For example, bash has a `globstar` option that enables the `**` metacharacter.

[^3]: I'm cheating a bit here. `[` is actually a shell built-in, but there's also a program with the same name and functionality, similar to `echo`. You can check this with `type [`.
