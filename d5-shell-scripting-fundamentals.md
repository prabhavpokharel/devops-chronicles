# Shell Scripting Fundamentals

## Why This Matters

Everything up to this point has been typing commands one at a time. A script is just a way of writing a whole sequence of those commands down once, so the machine can run them in order without you sitting there typing each one. This is the actual bridge between "I know Linux commands" and "I can automate things" — which is most of what a DevOps engineer's day-to-day work actually is.

---

## The Shebang Line: `#!/bin/bash`

Every proper shell script starts with a line like this, as the very first line of the file:

```bash
#!/bin/bash
```

This is called a **shebang**. It looks like a comment (lines starting with `#` are normally ignored), but the kernel treats this specific pattern as a special instruction: it tells the system exactly which program should be used to interpret and run the rest of the file. `#!/bin/bash` says "run everything below this line using the Bash interpreter, located at `/bin/bash`."

Without a shebang, the system has no reliable way of knowing whether a file full of text is meant to be a Bash script, a Python script, or something else entirely — it would have to guess, or you'd have to specify the interpreter manually every single time you ran it.

This connects to something worth understanding by contrast: a compiled language like C doesn't need any of this. A C program gets compiled directly into machine code — a binary the processor can execute natively, with no interpreter involved. A shell script, by contrast, stays as plain text forever; the shebang is what tells the kernel which interpreter to hand that text to, every time it runs.

---

## Two Different Curly/Parenthesis Syntaxes — Don't Mix Them Up

Bash scripting leans heavily on two syntax forms that look similar but do genuinely different jobs:

### `$(...)` — Command Substitution

This runs whatever command is inside the parentheses, and substitutes its **output** directly into the surrounding text. Think of it as "run this command right here, and use whatever it prints as if I'd typed that instead."

```bash
echo "Today is: $(date)"
```

Here, `$(date)` runs the `date` command, captures its output, and drops that output directly into the string being echoed.

### `${...}` — Variable Expansion

This refers to the **value of a variable**, not the output of a command. The curly braces aren't strictly required for simple cases (`$VAR` often works fine on its own), but they matter once a variable name needs to be clearly separated from surrounding text — for instance, `${VAR}name` unambiguously means "the value of VAR, followed by the literal text 'name'," whereas `$VARname` would be misread as a single variable called `VARname` that doesn't exist.

The convention, and it is a genuinely strong convention rather than a strict rule, is that **variable names are written in capital letters** — `NAME`, `MY_VAR`, `COUNT` — which makes them visually distinct from commands and regular text at a glance.

---

## `echo` and `date`

Two of the most-used commands in any script.

`echo` prints text to the screen (or wherever its output is redirected to). The examples from class:

```bash
echo 'Hello, Linux'
echo "Today is: $(date)"
echo "you are running as: $(whoami)"
```

The first uses single quotes, which take everything inside completely literally — no substitution happens inside single quotes at all. The second and third use double quotes, which *do* allow command substitution (`$(...)`) and variable expansion (`${...}`) to happen inside them. This distinction — single quotes preserve text exactly as-is, double quotes still allow substitution — comes up constantly once scripts get more complex.

`date` simply prints the current date and time. On its own it's not that interesting, but wrapped in `$(date)` inside a larger string, it becomes a way to timestamp output — genuinely common in real scripts, especially logging.

---

## `chmod` — Changing Permissions

`chmod` modifies who can read, write, or execute a file or directory. There are two ways to express the change: symbolic (letters) and numeric (a three-digit shortcut). Both do the same underlying thing; numeric is faster once it's familiar.

### Symbolic mode

```bash
chmod +x filename          # equivalent to: chmod u+x filename
chmod u+rwx filename       # the current user (owner) gets read, write, and execute
chmod go+r filename        # group and others both get read permission
```

`u`, `g`, `o` stand for **user** (the owner), **group**, and **others** — the same three categories from the `ls -la` permission string covered in earlier notes. `+` adds a permission, `-` would remove one, and `=` would set it exactly (overwriting whatever was there).

### Numeric mode

Each permission has a fixed point value:

| Permission | Value |
| --- | --- |
| Read | 4 |
| Write | 2 |
| Execute | 1 |

Adding the relevant values together gives a single digit representing all three permissions for one category. Three digits, always in the order **User, Group, Other**, cover the whole file:

```bash
chmod 777 filename   # U, G, and O all get everything: 4+2+1 = 7 for each
chmod 700 filename   # U gets everything (7); G and O get nothing (0)
chmod 544 filename   # U gets read+execute (4+1=5); G and O get read only (4)
chmod 633 filename   # U gets read+write (4+2=6); G and O get write+execute (2+1=3)
```

That last example is worth pausing on, since it's a slightly unusual combination: giving group and others *write and execute* but not *read* is technically valid but rarely what you actually want in practice — worth remembering the numbers are just arithmetic, not a guarantee the combination makes sense for real use.

---

## Running a Script: `bash file.sh` vs `sh file.sh`

Once a script exists, there are a few ways to actually run it:

```bash
bash file.sh
sh file.sh
```

Both of these work by explicitly naming the interpreter up front and handing the script to it — which means the file doesn't even need execute permission or a shebang line for this specific method to work, since you're telling the shell directly which interpreter to use.

The alternative, `./file.sh`, relies entirely on the shebang line and the file actually having execute permission (`chmod +x file.sh` first) — the kernel reads the shebang itself to figure out the interpreter, rather than you specifying it on the command line.

---

## `chown` — Changing Ownership

Where `chmod` changes *what's allowed*, `chown` changes *who owns it* — both the user and/or the group.

```bash
sudo chown ownername:groupname filename
```

The general syntax: everything before the colon changes the owning user, everything after the colon changes the owning group. Either half can be omitted to leave that part unchanged:

```bash
sudo chown :ollamagrp test.sh   # only the group changes, to ollamagrp; owner stays the same
sudo chown ollama test.sh       # only the owner changes, to ollama; group stays the same
```

`chown` almost always needs `sudo`, since changing who owns a file is inherently a privileged operation — otherwise anyone could just hand ownership of files to themselves.

---

## Conditionals

Bash's `if` structure follows a consistent shape, always closed with `fi` (which is just "if" spelled backward — a Bash convention for closing several kinds of blocks).

**Simple if:**

```bash
if [ condition ]
then
    echo "condition was true"
fi
```

**If / else:**

```bash
if [ condition ]
then
    echo "condition was true"
else
    echo "condition was false"
fi
```

**If / elif / else — for checking multiple possibilities in sequence:**

```bash
if [ condition_one ]
then
    echo "first case"
elif [ condition_two ]
then
    echo "second case"
else
    echo "neither matched"
fi
```

`elif` is Bash's version of "else if" — it lets you check additional conditions in sequence without nesting a new `if` block inside the `else` of the previous one.

### Conditional Operators for Testing Files and Strings

Inside the square brackets, these operators let a condition check something real about the filesystem or a variable, rather than just comparing numbers or text directly:

| Operator | Type | What it checks | Example |
| --- | --- | --- | --- |
| `-f` | File test | True if the target exists and is a regular file | `[ -f "/etc/passwd" ]` |
| `-d` | File test | True if the target exists and is a directory | `[ -d "/var/log" ]` |
| `-z` | String test | True if the string is zero-length (empty) | `[ -z "$MY_VAR" ]` |

`-z` in particular comes up constantly for defensive scripting — checking whether a variable was actually set to something before a script tries to use it, to avoid errors further down.

---

## Task 1: IFS (Internal Field Separator)

`IFS` is a special shell variable that controls **where Bash splits input into separate pieces** — words, fields, columns, however you want to think about it. Its default value, even though it's invisible, is space, tab, and newline combined — which is exactly why typing several words separated by spaces normally gets treated as separate arguments without you ever having to configure anything.

Setting `IFS` explicitly changes that splitting behavior. A concrete example:

```bash
IFS=:
read name age city <<< "Ram:20:Kathmandu"
```

Result:

```text
name = Ram
age  = 20
city = Kathmandu
```

Here, setting `IFS=:` tells Bash that colons, not spaces, are what separate one field from the next. `read` then takes the input string and assigns each colon-separated piece to the next variable name listed — `name`, then `age`, then `city`, in order.

**What IFS is actually useful for:**

- Splitting strings into individual fields, using whatever delimiter the data actually uses
- Reading delimiter-separated data — CSV-like text, colon-separated files like `/etc/passwd`, and similar formats
- Processing a file's contents one line at a time
- Turning a delimited string into something closer to an array, one variable per field

The short version: **IFS tells the shell exactly where one field ends and the next one begins**, and changing it lets a script parse data that isn't naturally space-separated.

---

## Task 2: Reading a File Line by Line

The assigned task was to read `/etc/hostname` and print each line, prefixed with `Line:`. If the file has 5 lines, the loop runs exactly 5 times.

```bash
while IFS= read -r LINE
do
    echo "Line: $LINE"
done < /etc/hostname
```

Breaking down exactly what each piece is doing, since every part here is pulling weight:

- **`done < /etc/hostname`** — this is what feeds the entire file into the loop as input in the first place. The `<` redirects the contents of `/etc/hostname` to become the input stream that `read` pulls from, one line at a time, for the whole duration of the loop.
- **`read -r LINE`** — reads a single line from that input stream, up until it hits a newline character, and stores whatever text it read into the variable `LINE`. The `-r` flag is a small but important detail: without it, `read` treats a backslash (`\`) inside the line as an escape character and may alter the text; `-r` tells it to read the line completely raw, exactly as it appears in the file.
- **`IFS=`** — set to nothing at all, deliberately, right before the `read`. This temporarily disables Bash's usual field-splitting behavior for this read, so that leading or trailing whitespace on each line is preserved exactly rather than being silently trimmed away.
- **`echo "Line: $LINE"`** — runs once per loop iteration, printing the fixed text `Line:` followed by whatever that specific line actually contained.
- **`while ... ; do`** — the loop itself keeps running for as long as `read` successfully manages to pull another line from the input. The moment `read` hits the end of the file and there's nothing left to read, it fails to read a new line, and that failure is exactly what causes the `while` loop to exit naturally — no explicit counter or line-count check needed anywhere in the script.

This pattern — `while IFS= read -r LINE; do ... done < file` — is close to the standard, idiomatic way to process a file line by line in Bash, and it's worth memorizing close to verbatim rather than reconstructing it from scratch each time, since the small details (`-r`, the empty `IFS=`) are easy to forget and each one fixes a specific, real edge case.
