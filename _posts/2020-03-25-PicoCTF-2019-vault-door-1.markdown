---
layout: post
title:  "PicoCTF 2019 - vault-door-1"
date:   2020-03-25 19:46:39 +1100
categories: pico ctf picoctf
---

## vault-door-1
- Points: 100
- Category: Reverse Engineering

**Description/Question:**

```
This vault uses some complicated arrays! I hope you can make sense of it, special agent. The source code for this vault is here: VaultDoor1.java
```

**Solution**

This is the second challenge in the reverse engineering series in PicoCTF called vault-doors.

We are given a Java source code file which we need to look at to find the flag.

{% highlight java %}
import java.util.*;

class VaultDoor1 {
    public static void main(String args[]) {
        VaultDoor1 vaultDoor = new VaultDoor1();
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter vault password: ");
	String userInput = scanner.next();
	String input = userInput.substring("picoCTF{".length(),userInput.length()-1);
	if (vaultDoor.checkPassword(input)) {
	    System.out.println("Access granted.");
	} else {
	    System.out.println("Access denied!");
	}
    }

    // I came up with a more secure way to check the password without putting
    // the password itself in the source code. I think this is going to be
    // UNHACKABLE!! I hope Dr. Evil agrees...
    //
    // -Minion #8728
    public boolean checkPassword(String password) {
        return password.length() == 32 &&
               password.charAt(0)  == 'd' &&
               password.charAt(29) == '7' &&
               password.charAt(4)  == 'r' &&
               password.charAt(2)  == '5' &&
               password.charAt(23) == 'r' &&
               password.charAt(3)  == 'c' &&
               password.charAt(17) == '4' &&
               password.charAt(1)  == '3' &&
               password.charAt(7)  == 'b' &&
               password.charAt(10) == '_' &&
               password.charAt(5)  == '4' &&
               password.charAt(9)  == '3' &&
               password.charAt(11) == 't' &&
               password.charAt(15) == 'c' &&
               password.charAt(8)  == 'l' &&
               password.charAt(12) == 'H' &&
               password.charAt(20) == 'c' &&
               password.charAt(14) == '_' &&
               password.charAt(6)  == 'm' &&
               password.charAt(24) == '5' &&
               password.charAt(18) == 'r' &&
               password.charAt(13) == '3' &&
               password.charAt(19) == '4' &&
               password.charAt(21) == 'T' &&
               password.charAt(16) == 'H' &&
               password.charAt(27) == '1' &&
               password.charAt(30) == 'f' &&
               password.charAt(25) == '_' &&
               password.charAt(22) == '3' &&
               password.charAt(28) == 'e' &&
               password.charAt(26) == '5' &&
               password.charAt(31) == 'd';
    }
}
{% endhighlight %}

This challenge is similar to the previous vault door training challenge except the password match check is split into individual characters and then checked out of order.

All we need to do is order each of the individual characters in the `checkPassword` function.

I've managed to do this with some really disgusting bash.

{% highlight bash %}
cat VaultDoor1.java |grep charAt |sed "s/.*charAt(//g; s/).*== '/ /g; s/' \&\&//g; s/';//g" |sort -n |cut -d' ' -f2 |sed ':a;N;$! ba;s/\n//g'
{% endhighlight %}

I'll try to step through it so it makes sense.

- read the source code with `cat`
- use `grep` to find lines with "charAt" to only have the lines with the characters we want
- `sed` is used to replace anything matching regex `.*charAt(` with nothing
- another `sed` command to replace anything matching regex `).*== '` with a space
- another 2 `sed` commands to remove `' &&` or `';`

This will leave us with:
```
0 d
29 7
4 r
2 5
23 r
3 c
17 4
1 3
7 b
10 _
5 4
9 3
11 t
15 c
8 l
12 H
20 c
14 _
6 m
24 5
18 r
13 3
19 4
21 T
16 H
27 1
30 f
25 _
22 3
28 e
26 5
31 d
```

This is each character with it's index on the left. Next we can `sort` them numerically with `sort -n`.
Then `cut` is used to split each line by the space `cut -d' '` and only display the value on the right/second value `cut -d' ' -f2`.
Lastly we use `sed` again to remove any new line characters `sed ':a;N;$! ba;s/\n//g'`.

The result is `d35cr4mbl3_tH3_cH4r4cT3r5_51e7fd` which means our flag is `picoCTF{d35cr4mbl3_tH3_cH4r4cT3r5_51e7fd}`.
