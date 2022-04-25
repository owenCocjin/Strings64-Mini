# Strings64 One-Liner
> Deconstruction of my Python strings clone (with auto base64 decoding) as best I can

## Intro
I was doing some PicoCTFs and though it might make my life easier to make a strings clone that automatically decodes base64 strings whenever it can. "Strings" is a common Linux command that will print out all consecutive ASCII characters in a file. Once the clone was done, I challenged myself to do it in as few lines as possible. As far as I know, one line is the smallest you can get.

In this file, I'll go over the whole process in steps:
1. Note goals and rules
2. Note basic requirements/restrictions of the code
3. Explain broadly how the overall flow works
4. Review the main tools/Python functionalities used to achieve this one-liner
5. Dissect the code

---

## 1. Goals & Rules
The goal was to create a Python "strings" command clone that would automatically decode base64 strings wherever it could, in as few lines as possible. As you may know, Python is space delimited. Meaning instead of defining loops and functions with curly brackets, it uses line indents. This means we can't define functions, use if statements, or use while/for loops in our one liner how you normally would, so we'll need to take this into consideration.

Here was the list of requirements for this project:
- Take a file name as argument from the command line
- Take an optional minimum string length from the command line
- Print all strings >= to that minimum length
- Decode all strings from base64 when possible, and display in an appropriate manner. In this case, I print the decoded bytes in green beside the original string.
- No semicolons (more details below)!

#### About Semicolons
Python doesn't require it, but it is possible to use semicolons to define multiple lines. For example, the following:
```
x=5
print(x)
```
Can be written in one line using semicolons:
```
x=5; print(x)
```
I considered this cheating as it essentially defeats the purpose of this challenge. So no semicolons were used!

---

## 2. Code Requirements & Restrictions
This code is by no mean efficient. It was meant to be a challenge against my Python skills and knowledge.

For restrictions, there were some challenges with input validation from the user (check the file exists, passing a valid string length, etc...) but these have been addressed, as you'll see later.
Because we use walrus operators (see below), we must use Python >= 3.8 as they were introduced in this version.
We also use ANSI colours to print the base64 decoded string, so an ANSI enabled terminal (like Bash) is strongly recommended, but technically not required.

---

## 3. Flow Overview
The flow can be broken into 2 general blocks, with 2 blocks under the second:
1. Setup variables and imports
2. Loop through the file one byte at a time and:
<ol type='a'>
	<li>Process the current string and print if needed</li>
	<li>Try to decode current string from base64 and print if successful</li>
</ol>

---

## 4. Python Requirements
There are 3 main python functionalities that really enable this project:
- [Walrus operator](https://docs.python.org/3/whatsnew/3.8.html) : `(x:=5)`
- [Ternary operator](https://python-reference.readthedocs.io/en/latest/docs/operators/ternary.html): `x=5 if True else 4`
- [List comprehension](https://docs.python.org/3/tutorial/datastructures.html#tut-listcomps): `x=[i for i in [1,5,3,9] if i>4]`
> The walrus operator is introduces in Python 3.8, therefore the minimum version to get this working is Python 3.8

#### Walrus Operator
The walrus operator allows us to assign a variable in-place, and return it's value. This is similar to how you can assign and evaluate a variable in place in C:
```
#include <stdio.h>

int x,y;

void main(){
	if ((x=5)==5){  //This sets x to 5 and returns 5
		printf("x is %d\ny is %d\n",x,(y=7));  //Here we set y to 7 and return it's value
	}//if()
}//main()
```
In Python, the walrus operator can be expanded to something similar. Instead of doing this:
```
x=5
y=7
if x==5:
	print(f"x is {x}\ny is {y}")
```
We can do the following:
```
if (x:=5)==5:  #Set x to 5 and compare it's value to 5
	print(f"x is {x}\ny is {(y:=7)}")  #Set y to 7 and print it's value
```
This not only saves us multiple lines, it allows us to set variables while evaluating them at the same time.

#### Ternary Operator
The ternary operator allows us to smush a 4 line if-else statement into just 1. The ternary operator is a bit jarring to understand at first, but I'll try to explain it here.
Take the following if-else:
```
if True:
	x="Right"
else
	x="Wrong"
```
We can re-wright it as a ternary operation:
```
x=("Right" if True else "Wrong")
```
Wait! Why are we only setting x once? This is because we aren't __SETTING__ x if True, but we're determining what the __VALUE__ is to be set! The parenthesis help clarify things here, where we're setting the variable "x" to: `("Right" if True, "Wrong" otherwise)`.

One more important thing to note about ternary is the order the interpreter evaluates the expression. Using this as example: `var=(x if y else z)` the interpreter evaluates the variables in this order:
1. `y`
2. `x`
3. `z`
4. `var`

This is because the interpreter must determine if `y` is True first, then evaluate `x` if so. If not, evaluate `z`. After we evaluate the parenthesis, assign `var` whatever was returned by parenthesis.
This is extremely important to the success of this one-liner for cases where we might try to interpret a variable before it even exists. It also allows us to essentially error check variables. We'll go into this while dissecting the code.

### List Comprehension
List comprehension allows for a simple way to do simple processing/creation of a list. A very basic example to create a list of ints from 10 to 20:
```
int_list=[i for i in range(10,21)]
#Result: [10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20]
```
This is similar to:
```
int_list=[]
for i in range(10,21):
	int_list.append(i)
```

We can also process the variable "i" before returning it. Let's say we want to get mod 5 of each int from 10 to 20:
```
mod5_list=[i%5 for i in range(10,21)]
#Result: [0, 1, 2, 3, 4, 0, 1, 2, 3, 4, 0]
```
But what is we **ONLY** wanted a list of ints that are multiples of 3? We can use an if statement in the list comprehension:
```
only_5_multiple=[i for i in range(10,21) if i%3==0]
#Result: [12, 15, 18]
```
This only returns `i` if i%3 is equal to 0

---

## 5. Code Dissection
Finally,we will start to dissect the code. Again, we can break it into blocks:
```
One-line
  |_ Variable assigning & imports
  |_ Main loop
     |_ Print string
     |_ Print base64 decoded
```
We will analyze each block independently.
Here is the full code for reference:
![Full code snippet](/Pics/fullcode.png)
```
[(str_buff:=''),(ok_chars:="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/="),(base64:=__import__("base64")),(sys:=__import__("sys")),(path:=__import__("os").path),(str_len:=int(sys.argv[2]) if len(sys.argv)>2 and sys.argv[2].isdecimal() else 4),(eoe:=lambda:end+1 if len(str_buff)>((end:=str_buff.find('=')+1) if str_buff.find('=')>-1 else (end:=len(str_buff))) and str_buff[end]=='=' else end)]+[(str_buff:=chr(bool(print((str_buff+("  \033[92m["+f"{base64.b64decode(str_buff)}"[2:-1]+"]\033[0m" if len(str_buff)>=str_len and any((not len([c for c in str_buff[:eoe()] if c in ok_chars])%4, not len([c for c in str_buff[:eoe()] if c in ok_chars[:-1]])%4)) else '')+'\n') if len(str_buff)>=str_len else '',end='')))[0:0] if not 32<=menu<=126 else str_buff+chr(menu)) for menu in open((sys.argv[1] if len(sys.argv)>1 and path.exists(sys.argv[1]) else print("""string64.mini.py <file> [size]\nPrint all strings in a file, and base64 decode when possible\n  <file>: Target file\n  [size]: Min string length""")+exit(1)),'rb').read()+b'\x00']
```

### Variable Assigning & Imports
This is the first block:
![Var assigning and imports](/Pics/block1.png)
```
[(str_buff:=''),(ok_chars:="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/="),(base64:=__import__("base64")),(sys:=__import__("sys")),(path:=__import__("os").path),(str_len:=int(sys.argv[2]) if len(sys.argv)>2 and sys.argv[2].isdecimal() else 4)]
```
The block can be split like so:
```
[
	(str_buff:=''),
	(ok_chars:="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/="),
	(base64:=__import__("base64")),
	(sys:=__import__("sys")),
	(path:=__import__("os").path),
	(str_len:=int(sys.argv[2]) if len(sys.argv)>2 and sys.argv[2].isdecimal() else 4)
]
```
Due to how block 2 was created (see later), we have to wrap block 1 in a list. This list was simply "added" to the second list, even though neither of these lists themselves are used.
The key to this block was to use a walrus operator for each variable. `str_buff` is used to hold the current string (that is, all current consecutive printable ASCII characters in a row). `ok_chars` holds a list of base64 acceptable characters.

Next, we import 3 builtin libraries:
- **base64** is (obviously) used to decode a given string as best it can.
- **sys** is used to get command line arguments, such as the target file and the minimum string length.
- **os.path** helps check that the given file exists, allowing the script to avoid any file related errors.

As you may have noticed, we don't use the usual "import" keyword. Python doesn't allow us to use it how you might use a function. Instead, we can use the `__import__()` function. This takes the name of the lib to import as a string. It return the lib as a whole, so we need to assign the returned lib to a variable which we'll name after the imported library. Surprise! We do this with the walrus operator. For the `os` import we only need a specific submodule, so after calling `__import__()` we use `.path` to get the "path" submodule.

**After** importing the libraries, we set the minimum string length. Because this can be set by the user via cmdline argument, we must get it from `sys.argv`. There are 2 challenges with this:
1. Check that the user actually sets it and doesn't leave it blank
2. Check that the given value is actually an int
> For simplicity, the 2nd cmdline argument is the min string length.

`sys.argv` holds the command line arguments as a list of strings. `sys.argv[0]` is always the command name.

The solution I came up with is to use a ternary expression to check that `sys.argv`'s length is greater than 2 (the command name and the target file path), and check that `sys.argv[2]` is an integer. To do this, Python has a builtin function in all strings: `isdecimal()`. This returns True if "all characters in the string are decimal and there is at least one character in the string", as per `help(''.isdecimal)`. If both these conditions are met, we can safely convert `sys.argv[2]` to an int and store it to variable `str_len`.

![Check args length](/Pics/str_len_con1.png)
> _Checking `sys.argv` length_

![Check arg is an int](/Pics/str_len_con2.png)
> _Checking `sys.argv[2]` is an int_

The final step in block 1 is to create a function to find the 1st or 2nd (if it exists) '=' sign in the string if any. I've found that there are some cases where if a base64 strings ends with an equal sign followed by some acceptable characters (a-z, +, /), depending on the length it may not try to base64 decode the string. Broadly, we find the position of the first '=', or return the length of the string if one doesn't exist. Then we add 1 to the retrieved position only if the next character exists and it's an equal sign, else we return whatever value we already have.
We can use Python's "lambda" to create a function definition in-place, then assign it to a variable (`eoe` in this case) so we can call it like a normal function later on.

![Find equal sign function](/Pics/eoe_def.png)
> _The `eoe()` function definition_

The lambda definition can be expanded to:
```
def eoe():
	#Get position of first equal sign,
	#or length of str_buff
	if str_buff.find('=')>-1:
		end=str_buff.find('=')+1
	else:
		end=len(str_buff)

	#Return position of second equal sign if it exists
	if len(str_buff)>end and str_buff[end]=='=':
		return end+1
	return end
```

### Main Loop
This is the main loop block:
![Main loop block](/Pics/block2.png)
```
[(str_buff:=chr(bool(print((str_buff+("  \033[92m["+f"{base64.b64decode(str_buff)}"[2:-1]+"]\033[0m" if len(str_buff)>=str_len and any((not len([c for c in str_buff if c in ok_chars])%4, not len([c for c in str_buff if c in ok_chars[:-1]])%4)) else '')+'\n') if len(str_buff)>=str_len else '',end='')))[0:0] if not 32<=menu<=126 else str_buff+chr(menu)) for menu in open((sys.argv[1] if len(sys.argv)>1 and path.exists(sys.argv[1]) else print("""string64.mini.py <file> [size]\nPrint all strings in a file, and base64 decode when possible\n  <file>: Target file\n  [size]: Min string length""")+exit(1)),'rb').read()+b'\x00']
```
This block is used to loop through the target file. Originally, this used a while loop where as long as we can still read a byte from the target file, we continue the loop. To my knowledge, there is no way to put a while loop in one line. There _is_, however, a way to put a for loop in one line: list comprehension! Instead of looping while the currently read byte is not empty, we will read the entire file into memory, then loop through each byte. For each loop we will "evaluate" the current byte (really we set str_buff, but we will go into this in a bit).

Before we can do any of the above we need to make sure the file exists, then read it into memory. We do this by opening `sys.argv[1]` as a file only if the length of `sys.argv` is greater than 1 (the script name), and the given path exists. We confirm the path exists with `path.exists`. If at least one of these conditions aren't met, we will print a small help menu and exit. Because `print()` and `exit()` are both functions, we can execute them in-line and add their results together. Sice the `exit()` function exits immediately, it really doesn't matter that you can't add anything to None (`print()` returns None).
> Remember: We imported "os.path" into the variable "path" in block 1

After we read in the whole file, we append a null byte (`\x00`) to whole file. The way the string is processed is if it encounters a printable character it just adds it to the current `str_buff`, otherwise it'll print the string and try decoding it. If the target file ends with a printable string, it won't be printed! Adding the `\x00` doesn't harm the contents of the file, and will print any strings that end the file.

While looping through a bytes object, Python will return the int value of the current byte. For example, looping through `b'Hey'` using list comprehension will return a list of ints:
```
target=b'Hey'
print([cur_byte for cur_byte in target])
#Result: [72, 101, 121]
```
While looping through the file, we set the variable `menu` to the current byte as an int. This means we must be mindful that `menu` will be an int and **NOT** a bytes object or string.

![Whole for block](/Pics/for_whole_block.png)
> _for loop block (processing truncated)_

![For block expanded](/Pics/for_loop_equivalent.png)
> _for loop block equivalent code_

### Processing a Character
This is the block that processes the current byte and prints the string is applicable:
![String processing block](/Pics/str_processing_block.png)
```
(str_buff:=chr(bool(print((str_buff+("  \033[92m["+f"{base64.b64decode(str_buff)}"[2:-1]+"]\033[0m" if len(str_buff)>=str_len and any((not len([c for c in str_buff if c in ok_chars])%4, not len([c for c in str_buff if c in ok_chars[:-1]])%4)) else '')+'\n') if len(str_buff)>=str_len else '',end='')))[0:0] if not 32<=menu<=126 else str_buff+chr(menu))
```

In the for loop, we read each character into the variable "menu". We use a ternary operator that will set `str_buff` to `str_buff+chr(menu)` if `menu` is between 32 and 126 (inclusive). This is the ASCII printable character range. Below is the expanded equivalent of the string processing block:

![String processing expanded](/Pics/str_processing_expanded.png)
> _String processing equivalent, with base64 decoding removed (for readability)_

We use a walrus operator that sets `str_buff` to the results of a ternary operator that says: "If `menu` is **NOT** a printable ASCII char, print `str_buff` then return an empty string, otherwise return `str_buff` + `chr(menu)`". Remember: a ternary operator is used to return a value if x, else return y. Here we either return an empty string (equivalent to clearing `str_buff`), or return `str_buff+chr(menu)` (equvalent to appending `chr(menu)` to `str_buff`).

The "else" statement is the easiest to understand as it just returns `str_buff` plus menu.

The "if" is a bit trickier:
![String processing "if" block](/Pics/str_processing_if.png)
```
chr(bool(print((str_buff+BASE64_PROCESSING+'\n') if len(str_buff)>=str_len else '',end='')))[0:0]
```
We can expand this chunk into:
```
if len(str_buff)>=str_len:
	to_print=(str_buff+BASE64_PROCESSING+'\n')
else:
	to_print=''

chr(
 bool(
  print(to_print)
 )
)[0:0]
```
We ultimately return an empty string because we slice the character from position 0 to position 0. This returns an empty string no matter the contents. We convert the print to a bool because `print()` returns `None`, which can't be appended to a string and can't be converted to a char (to then be appended to `str_buff`). By converting `None` to a bool, we convert it to 0, which can then be converted to the ASCII char of `'\x00'`. Again, it doesn't matter if this is a valid/printable character because we convert it to an empty string.

In the print statement, we print the current "str_buff", plus some base64 decoding content that is discussed in the next section, then a newline.

To sum this block up, we either:
- Print str_buff+base64 decoded content+newline, then clear `str_buff`
- Append the current byte as a character to `str_buff`

### Base64 Processing
At this point, the script will print all strings, similar to the `strings` Linux command, but we want to try to automatically decode any base64 we see. The "base64" Python library does a great job at cleaning the data (removing extra non-base64 chars), but we still need to do some light processing; We need to ensure the length is a multiple of 4, and only try to decode base64 up to either '=' or "==", if either exists.

![Base64 string processing](/Pics/base64_processing.png)
```
("  \033[92m["+f"{base64.b64decode(str_buff)}"[2:-1]+"]\033[0m" if len(str_buff)>=str_len and any((not len([c for c in str_buff[:eoe()] if c in ok_chars])%4, not len([c for c in str_buff[:eoe()] if c in ok_chars[:-1]])%4)) else '')
```
The expanded equivalent is:
```
#Make sure the number of valid chars is a multiple of 4
ok_char_len_1=not len([c for c in str_buff[:eoe()] if c in ok_chars])%4
#Same as above, but exclude '=' from check
ok_char_len_2=not len([c for c in str_buff[:eoe()] if c in ok_chars[:-1]])%4

#Only try to base64 decode if the length is greater than the min length,
#and the number of ok_chars with ot without '=' is a multiple of 4
if len(str_buff)>=str_len and any((ok_chars_len_1, ok_chars_len_1)):
	decoded=base64.b64decode(str_buff)
	print(f"  \033[92m[{decoded}\033[0m")
else:
	print('')
```

Because we can't wrap anything if `try/except`, we need to validate data before sending it to base64 decoding. There are 2 main issues we need to look out for:
- `base64.b64decode()` will throw errors if the number of data characters is not a multiple of 4
- The string is of greater length than the defined minimum
These are addressed as 2 separate checks in the ternary operation.

The latter is achieved by confirming the length of `str_buff` is greater or equal to the min:

![Base64 check min length](/Pics/base64_check_len.png)
> _Check the string length_

The former is completed by using list comprehension to loop through `str_buff`, creating a list of characters that are in the `ok_chars` list (that being, they are base64 characters). If there is 1 or 2 consecutive '=' in the string, we slice `str_buff` to exclude anything after that to decrease our chances of a false negative. We use the `eoe()` function defined earlier to get the position we need to slice at. We then get the length of this list and return the boolean opposite of the length mod 4. We get the boolean opposite because if the length _IS_ a multiple of 4, it will return 0 (boolean False). Repeat the above process against the list of `ok_chars` without the '=', then make sure either of these succeeded using Python's `any()` function. This returns True if any value in an iterable is a boolean True.

If both these checks pass as True, we can then try decoding the string without fear the script will crash. I elected to wrap the decoded string in square brackets and print it green (in ANSI) to make it more distinguishable from the original string.

---

## Conclusion
That is the code explained to the best of my abilities! Hopefully that all made sense. The main takeaways from all of this are that there are always multiple solutions when it comes to coding, and it's really up to you and your skills to determine which one you take.

This project was by no mean meant to be efficient. I though it would make for a fun challenge that could demonstrate some out-of-the-box thinking, and maybe help inspire others to pursue similar problems themselves.
