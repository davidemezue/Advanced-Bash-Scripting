# Advanced Bash Scripting

## Lab Objective
Learn how to make bash scripts take decisions using if, elif, else, file tests and command-line arguments.

## Lab Purpose
Bash (the Bourne Again Shell) is a Unix shell and command language, and a free replacement for the original Bourne shell. Many Linux distributions use it as their default shell, though recent Kali versions default to zsh. Scripts in this lab start with #!/bin/bash, so they run in bash either way.

## Lab Tool
- Kali Linux

## Lab Topology
A single Kali Linux VM. Work as a normal user, not root, because Task 3’s permission test needs it.

---

## Setup

Open a terminal and create a working folder:

```bash
mkdir ~/bashlab && cd ~/bashlab
```

Use straight quotes (") when typing scripts. Curly quotes (“ ”) copied from web pages break bash.

For every script: create it with nano scriptname.sh, save with Ctrl+O, Enter, exit with Ctrl+X, then run chmod +x scriptname.sh.

---

## Lab Walkthrough

### Task 1: Your First Conditional

An if statement has this structure:

```bash
if [ condition ]
then
    some action
else
    some other action
fi
```

Spaces are required just inside the brackets, and the statement must end with fi (if backwards).

Create sample07.sh:

```bash
#!/bin/bash
number=5

if [ "$number" -eq 5 ]
then
    echo "true"
else
    echo "false"
fi
```

Run it:

```bash
chmod +x sample07.sh
./sample07.sh
```

Expected output: true

**Comparison operators:**

| Type | Operator | Meaning |
|---|---|---|
| Numbers | -eq | equal to |
| Numbers | -ne | not equal to |
| Numbers | -gt | greater than |
| Numbers | -lt | less than |
| Numbers | -ge / -le | greater/less than or equal to |
| Strings | = / != | equal / not equal |

Try it: Change number=5 to number=7, run it again, and confirm the output is now false.

### Task 2: Accepting and Validating User Input

This script reads a number, rejects bad input, then uses elif to handle three cases.

Create sample08.sh:

**Input:**

```bash
#!/bin/bash
echo "Enter a whole number between 1 and 10:"
read value

# Reject anything that is not a whole number
if ! [[ "$value" =~ ^[0-9]+$ ]]
then
    echo "Error: '$value' is not a whole number."
    exit 1
fi

# Reject numbers outside the range
if [ "$value" -lt 1 ] || [ "$value" -gt 10 ]
then
    echo "Error: $value is outside the range 1-10."
    exit 1
fi

if [ "$value" -gt 5 ]
then
    echo "$value is greater than 5"
elif [ "$value" -eq 5 ]
then
    echo "$value is equal to 5"
else
    echo "$value is less than 5"
fi
```

Now run:

```bash
chmod +x sample08.sh
```

Then:

```bash
./sample08.sh
```

Run it several times with these inputs:

| Input | Expected output |
|---|---|
| 8 | 8 is greater than 5 |
| 5 | 5 is equal to 5 |
| 3 | 3 is less than 5 |
| abc | Error: 'abc' is not a whole number. |
| 11 | Error: 11 is outside the range 1-10. |
| (just press Enter) | Error: '' is not a whole number. |

After a run, echo $? prints the exit code: 0 means success, 1 means the script rejected the input.

**New concepts:**

- `[[ ... =~ ... ]]` matches text against a pattern (a regular expression). Here it checks that the input is all digits.
- `||` means OR. `&&` means AND.
- `exit 1` stops the script and reports an error.

### Task 3: Testing Files Safely

This script checks whether a file exists and is writable, then acts accordingly. It appends with >> instead of overwriting, so it never destroys existing data.

Create sample09.sh:

```bash
#!/bin/bash
echo "Please input the name of the file:"
read input

if [ -z "$input" ]
then
    echo "Error: no file name entered."
    exit 1
fi

if [ -f "$input" ] && [ -w "$input" ]
then
    echo "Yes" >> "$input"
    echo "File exists and is writable. Appended 'Yes' to $input"
elif [ -e "$input" ]
then
    echo "Error: $input exists but is not a writable regular file."
    exit 1
else
    touch "$input"
    echo "Yes" > "$input"
    echo "File did not exist. Created $input and wrote 'Yes' to it."
fi
```

**File test operators used:**

| Test | True when |
|---|---|
| -f | it exists and is a regular file |
| -w | it is writable by you |
| -e | it exists (file or directory) |
| -z | the string is empty |

Make it executable and run it:

```bash
chmod +x sample09.sh
./sample09.sh
```

**Run these tests in order:**

1. **New file.** Enter notes.txt. The script creates it. Check with `cat notes.txt`, which should show Yes.

2. **Existing file.** Run again with notes.txt. Check with `cat notes.txt`, which should now show Yes twice.

3. **Read-only file.** Run `./sample09.sh`, input notes.txt, then:

```bash
chmod 444 notes.txt
```

Then run the script with notes.txt. You should see the “not a writable regular file” error.

4. **Directory.** Run `mkdir testdir`, then run the script with testdir. You should see the same error.

5. **Empty input.** Press Enter without typing anything. You should see Error: no file name entered.

Clean up afterwards:

```bash
chmod 644 notes.txt && rm -rf notes.txt testdir
```

### Task 4: A Security Use Case: Host Availability Check

Administrators often automate checks like “is this host reachable?” This script takes the host as a command-line argument.

Create sample10.sh:

```bash
#!/bin/bash
# Usage: ./sample10.sh <host>

if [ "$#" -ne 1 ]
then
    echo "Usage: $0 <host>"
    exit 1
fi

host="$1"

if ping -c 1 -W 2 "$host" > /dev/null 2>&1
then
    echo "$(date '+%Y-%m-%d %H:%M:%S') $host is UP"
else
    echo "$(date '+%Y-%m-%d %H:%M:%S') $host is DOWN"
fi
```

Run these:

```bash
chmod +x sample10.sh
./sample10.sh                  # prints the Usage message
./sample10.sh 127.0.0.1        # prints "... 127.0.0.1 is UP"
./sample10.sh 203.0.113.99     # prints "... 203.0.113.99 is DOWN"
```

203.0.113.99 is a reserved documentation address, so it is safe to test against and should not respond. This can take a couple of seconds.

**New concepts:**

- `$#` is the number of arguments given, and `$1` is the first argument.
- `> /dev/null 2>&1` hides all of the command’s output, so the if only uses its success or failure.
- `$(date ...)` inserts a command’s output into text.

**Optional logging:** Run `./sample10.sh 127.0.0.1 >> hostcheck.log` a few times and view the results with:

```bash
cat hostcheck.log
```

---

## Conclusion

This lab explores advanced Bash scripting concepts, including conditional statements, input validation, file tests, command-line arguments, and automated host availability checks.
