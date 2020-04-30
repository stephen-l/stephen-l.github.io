---
layout: post
title:  "Flare-On 2019 - Memecat Battlestation"
date:   2020-04-08 18:23:10 +1100
categories: flare falreon memecat battlestation fireeye ctf
---

## Memecat Battlestation
- Category: Reverse Engineering

**Description/Question:**

```
Welcome to the Sixth Flare-On Challenge!

This is a simple game. Reverse engineer it to figure out what “weapon codes” you need to enter to defeat each of the two enemies and the victory screen will reveal the flag. Enter the flag here on this site to score and move on to the next level.

* This challenge is written in .NET. If you don’t already have a favorite .NET reverse engineering tool I recommend dnSpy
** If you already solved the full version of this game at our booth at BlackHat or the subsequent release on twitter, congratulations, enter the flag from the victory screen now to bypass this level.
```

**Solution**

This was the first challenge of the Flare-On 2019 reverse engineering CTF. It is a Windows .NET executable, and it appears to be a game where your cat must destroy other cats by entering weapon codes.

![programcode](/assets/flareon/2019/memecatbattlestation/stage1.png)

The challenge was originally released early for BlackHat Las Vegas 2019 with a third stage which used RC4 encryption, but this one in the challenge only contains 2 stages.

Since it's a .NET binary, we can analyse it in a .NET decompiler. I like to use ILSpy. When we open the binary in ILSpy we find 3 classes of interest; Program, Stage1Form and Stage2Form.

If we look at the Program class first we can get an idea of how this application flows.

![programcode](/assets/flareon/2019/memecatbattlestation/programcode.png)

Basically the logo is displayed, then Stage1Form is loaded, then if a weapon code is returned, display Stage2Form and if another weapon code is returned, then the victory form will be displayed. The final flag will be generated from the 2 weapon codes entered at each stage.

In the Stage1Form class there is a function called `FireButton_Click` which will run when the fire button has been clicked, it then has an if statement checking that the form's codeTextBox.Text is equal to `RAINBOW`. This will be our first weapon code.

![programcode](/assets/flareon/2019/memecatbattlestation/stage1code.png)

Next, Stage2Form has the same `FireButton_Click` function, but this time the if condition passes the input to the `isValidWeaponCode` function. This function will take each character in the text box and XOR it (the caret symbol ^ tells us this is an XOR function) with the value "A", then if each encoded character is equal to the encoded characters in the `SequenceEqual` function, it will return true.

An XOR is a bitwise operator or function that takes 2 inputs. Below is an example of 2 binary numbers being XOR'd. It can be simplified like this, if you convert 2 values to binary and align them, if each input bit is the same, the output will be 0. If each input bit is different, the output will be 1. Below we can see the first bits in input A and input B are 1, so the output is 0. Then the second bits from A and B are 1 and 0, so the output is 1.

input A |0|0|1|1|
input B |0|1|0|1|
output |0|1|1|0|

![programcode](/assets/flareon/2019/memecatbattlestation/stage2code.png)

To solve this we just need to XOR each of the already encoded values with "A". This can be done in CyberChef and gives us the result `Bagel_Cannon`.

![programcode](/assets/flareon/2019/memecatbattlestation/cyberchef.png)

Once this is entered, we get our flag `Kitteh_save_galixy@flare-on.com`.
