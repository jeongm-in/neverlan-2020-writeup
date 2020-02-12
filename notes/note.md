# NeverLAN CTF 2020 

## Reverse Engineer

### 0. 
I worked mostly on the reverse engineering challenges. 
The first one I solved was called "Reverse Engineer". 

### 1. 
Here is the challenge description. 
"This program seems to get stuck while running... Can you get it to continue past the broken function?" 
It has links to the binary, and I downloaded the first one. I'm not sure what the second one was, but I guess it was for Mac OS users. 

### 2. 
First let's see what's in the binary. 
File command gives some information, and actually running the binary results in segfault. 

### 3.
Open the file in gdb; and run it so that it crashes. 
Once the program segfaults, run `backtrace` (`bt` for short ) to see what was on the function frame. 
Here we see `foo()` and `main` so we know that `main()` called `foo()`, which segfaulted.

### 4.
So what is in `main()` exactly?
`disas main` reveals the instructions within the `main` function. 

### 5.
Now let's take a look at what is going on in the `print` and `foo` functions. 

`objdump -d revseng` 

objdump command will dump the whole binary out. Here I salvaged `main`, `foo`, and `print` function instructions. 

I think `foo` has some loop going on, and some syntax within it is causing segfault.
However, what is important here is the print function. We see a bunch of hex values, which look like ASCII. 
So let's take a look into the print function.

### 6. 
print starts with theses instructions. move some hex value to rbp. 
Turns out, these are 

### 7. 
ASCII value for flag{}. So I'm 80% sure print function will give us the flag. 

### 8. 
Now when we look more into the print function, we see `malloc` with 21 and more ASCII code...

### 9. 
Which turns out to be `w 3 c o` and so on.
We can simply salvage ASCII code from this function and get the flag, but that's no fun. 

### 10.
Let's see if we can bypass the `foo()` call to get the `print` function running. 
Going back to the main function, here is the block of interest. 

It seems like we have some local varaible, which is compared to 0 and depending on the result, `main` will call either `print` or `foo`.

Let's check what is in `-0x4(%rbp)`. 

### 11.
Inside gdb, set a breakpoint at `main`. Then run `info local`. The correct command is `info locals`, but I like typing less. 
Gdb tells us that we have a variable named x. Is this the one at `-0x4(%rbp)`?

### 12.
It sure is! The address checks out. 
Moving on to the next line, we have `0x79` assigned to our variable x. 
Now we have `x = 121`, and we are ready to compare it with 0. 


### 13. 
Here is the pseudocode of the block of interest. 
If `x > 0`, `foo()` will be called and program will segfault. 
If not, `print` will be called and we will get the flag. 

Let's change the value of x then. 
Do `set variable x=-1`. 
We can run `info locals` again to see if x really is -1 now. 


### 14. 
Let's step some more past the `cmp` instruction. Indeed, we bypassed `foo` and entered `print`. 
Continue to the end, and we have our flag! 

I think this is the intended way to solve this challenge, but there are many other ways to solve it. 

For example, you can run the print function directly, which is how I solved it originally. 

### 15. 
`call print()`, it's that simple. 

This was not too bad because it didn't take any arguments or involve any vulnerabilities such as stack overflow. 
But still it was fun looking into the binary and understanding the instructions. 

