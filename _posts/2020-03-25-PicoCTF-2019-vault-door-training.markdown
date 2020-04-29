---
layout: post
title:  "PicoCTF 2019 - vault-door-training"
date:   2020-03-25 19:46:39 +1100
categories: pico ctf picoctf
---

## vault-door-training
- Points: 50
- Category: Reverse Engineering

**Description/Question:**

```
Your mission is to enter Dr. Evil's laboratory and retrieve the blueprints for his Doomsday Project. The laboratory is protected by a series of locked vault doors. Each door is controlled by a computer and requires a password to open. Unfortunately, our undercover agents have not been able to obtain the secret passwords for the vault doors, but one of our junior agents obtained the source code for each vault's computer! You will need to read the source code for each level to figure out what the password is for that vault door. As a warmup, we have created a replica vault in our training facility. The source code for the training vault is here: VaultDoorTraining.java
```

**Solution**

This challenge is the start of a reverse engineering series in PicoCTF called vault-doors.

We are given a Java source code file which we need to look at to find the flag.

{% highlight java %}
import java.util.*;

class VaultDoorTraining {
    public static void main(String args[]) {
        VaultDoorTraining vaultDoor = new VaultDoorTraining();
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

    // The password is below. Is it safe to put the password in the source code?
    // What if somebody stole our source code? Then they would know what our
    // password is. Hmm... I will think of some ways to improve the security
    // on the other doors.
    //
    // -Minion #9567
    public boolean checkPassword(String password) {
        return password.equals("w4rm1ng_Up_w1tH_jAv4_c0b141c5e30");
    }
}
{% endhighlight %}

This is the first challenge, so it's relatively straight forward.

In the code, there is a Scanner object created to take user input when the application is run.

{% highlight java %}
Scanner scanner = new Scanner(System.in); 
System.out.print("Enter vault password: ");
String userInput = scanner.next();
{% endhighlight %}

Once something has been entered, it will take that string, find the string "picoCTF{" and grab every character after that up to the second last character.

{% highlight java %}
String input = userInput.substring("picoCTF{".length(),userInput.length()-1);
{% endhighlight %}

Then that string is passed to a function called `checkPassword`. If the function returns a boolean value of `true`, then we will get the message `Access granted.`

{% highlight java %}
if (vaultDoor.checkPassword(input)) {
    System.out.println("Access granted.");
} else {
    System.out.println("Access denied!");
}
{% endhighlight %}

This function takes the string as input and checks if the string matches `w4rm1ng_Up_w1tH_jAv4_c0b141c5e30` with the `equals` function.
If the strings match, then the function will return `true`.

{% highlight java %}
public boolean checkPassword(String password) {
    return password.equals("w4rm1ng_Up_w1tH_jAv4_c0b141c5e30");
}
{% endhighlight %}

This means that our flag should be `picoCTF{w4rm1ng_Up_w1tH_jAv4_c0b141c5e30}`.
