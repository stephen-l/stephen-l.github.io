---
layout: post
title:  "Flare-On 2019 - Overlong"
date:   2020-04-30 17:45:41 +1100
categories: flare falreon overlong fireeye ctf
---

## Overlong
- Category: Reverse Engineering

**Description/Question:**

```
The secret of this next challenge is cleverly hidden. However, with the right approach, finding the solution will not take an <b>overlong</b> amount of time.
```

**Solution**

This was the second challenge of the Flare-On 2019 reverse engineering CTF. It is a Windows executable, to start the analysis we'll run strings on the binary.

![strings](/assets/flareon/2019/overlong/strings.png)

It's instantly apparent that this binary has little to no strings that mean much to us, but we do know that MessageBoxA is likely called and the string `Output` is likely used.

We can disassemble the binary to see more of what's going on. I'm using Ghidra this time.

In Ghidra, we can see that there are only 3 functions in the Symbol Tree.

![functions](/assets/flareon/2019/overlong/functions.png)

We can start of at the entry to see what happens when this binary is first run.

![entry](/assets/flareon/2019/overlong/entry.png)

Ghidra's decompiled view on the right can help simplify the assembly in this function. Instantly we see that there is a call to `MessageBoxA` which takes 4 arguments.
We can visit the Microsoft documentation for this function to get more information.

![messageboxa](/assets/flareon/2019/overlong/messageboxa.png)

The arguments that mean something to us are the 2nd (lpText) and 3rd (lpCaption) which is the text in the Messagebox and the title. The title is a string which Ghidra has called `s_Output_00403000` and in the disassembled view it gives us that string `Output`.

The lpText argument, which in this case is `local_88`, is not as easy to work out just by looking at it but if we follow the code we can get an idea of how it is created. At the beginning of the function we see that `local_88` is initialised as a byte array of size 128, it is then passed into function `FUN_00401160()` as the first argument. The second argument passed into this function is a pointer to some data and the last is an interger value of `0x1c` or `28`.

If we look at the address of `DAT_00402008` we can see it is a byte array that is 176 bytes long of non-readable data, this might be some encoded text. We have a better idea of what is going on so we can start renaming some variables to make it easier to read.

![renamed](/assets/flareon/2019/overlong/renamed.png)

Now we have enough information to jump into `FUN_00401160()`. We can start off by renaming some of the parameters that we already know.

![function1](/assets/flareon/2019/overlong/function1.png)

If we have a high level look at this function we can see that `local_8` is equal to 0, an infinite while loop is created which will only be exited if `int_28` is less than or equal to `local_8` or if `bVar1` is equal to 0. There is also an incrementer for `local_8` written as `local_8 = local_8 + 1`. This indicates to me that `local_8` is a counter. This value is also the only value that can be returned from the function.

Within the loop there is another function that is called that takes our empty byte array and the potentially encoded byte array `FUN_00401000(bytearr_text,ptr_bytearr_encoded)`. If we peek at this function we see that there is a lot of arithmetic performed on the potentially encoded array, then having the result stored in `local_8` and then towards the end of the function, the value in `local_8` is placed at the byte array text location. This very much looks like a decoding function.

![function2](/assets/flareon/2019/overlong/function2.png)

if we head back to the while loop, we know that it will loop through this decoding function taking 1 encoded byte at a time for a total of 28 times which is defined in the entry function. Although, we know the encoded array is 176 bytes long and the text array that is being filled is 128 bytes long. So if we debug this program and patch the `0x1c` (28) parameter to 128 or `0x84`, we will decode all of the possible information.

So now we can open Overlong.exe in x64dbg and patch the push of `0x1c` to `0x84`, then run the application until the messagebox appears with some decoded text.

Debug the application and find `push 0x1c` where the `0x1c` value is placed on the stack for the function.

![debug](/assets/flareon/2019/overlong/debug.png)

Then we right click it -> binary -> edit and change 1C to 84.

![patch](/assets/flareon/2019/overlong/patch.png)

Finally we run the application and a messagebox will appear with all of our decoded data.

![flag](/assets/flareon/2019/overlong/flag.png)

Our flag is `I_a_M_t_h_e_e_n_C_o_D_i_n_g@flare-on.com`.
