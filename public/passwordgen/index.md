# Password Generator


## Introduction
Hello there👋, I thought of making a small project to better grasp the python programming language and at the same time make something relevant to what I like a mini cyber project.😊

So I thought what should I do?🤔 Ahaa! I know a small program that suggests better passwords than what most people come up with like `password`😒, `&lt;insert username&gt;&lt;some year&gt;`😒 or `iloveyou`😒 which are all very easy to guess and bruteforce. So I started grinding away as usual, let&#39;s check the simple password generator below.

## Program
This is a password generator program written in python. Few libraries were used in this program to simplify the program and decrease written lines of code as much as possible. First of all we import the necessary libraries. Random to generate random ints, and string to populate the data lower, upper, digits and symbols.
```python
import random
import string
```

We then ask for the user input, initiating the program and determining the length of the password.
```python
print(&#34;hello, welcome to password generator!&#34;)
length = int(input(&#34;\nEnter the length of the password:&#34;))
```

Assign variables containing the letters, nums, and symbols from library.
```python
lower = string.ascii_lowercase
upper = string.ascii_uppercase
nm = string.digits
symbols = string.punctuation
```

Append all variables to one variable, in addition to that we randomize the generated string of password saving it to a temporary variable. Finally we move it from temp to a password variable to be printed.
```python
all = lower &#43; upper &#43; nm &#43; symbols
temp = random.sample(all, length)
password = &#34;&#34;.join(temp)
print(password)
```

## Moral of the story
Hope this small program has peaked your interest to discover more and more importantly develop more complex passwords. At least next time you think about making a new account on some website remember to use something similar such a **Password Manager** or think of some hard password to guess😉. Until next time see ya👋. 

![heckerman](https://media.tenor.com/VrzXhtoSwcsAAAAd/hacker-typing.gif)


---

> Author: w01v3r  
> URL: http://example.com/passwordgen/  

