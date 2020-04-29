---
layout: post
title:  "PicoCTF 2019 - Glory of the Garden"
date:   2020-03-25 19:46:39 +1100
categories: pico ctf picoctf
---

## Glory of the Garden
- Points: 50
- Category: Forensics

**Description/Question:**

```
This garden contains more than it seems. You can also find the file in /problems/glory-of-the-garden_5_eeb712a9a3bc1998ffcd626af9d63f98 on the shell server.
```

**Solution**

In this challenge, we are given a file called "garden.jpg". It can be seen below.

![garden](/assets/picoctf/2019/garden/garden.jpg)

We know that it's an image file and it's a simple picture of a garden.

Running the `file` command on the file will give us some more information such as the image type. Even though it has a JPG extension, it is still good to double check.

```
$ file garden.jpg 
garden.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, baseline, precision 8, 2999x2249, frames 3
```

We now know it's definitely a JPEG. Something I like to check in JPEG images is the header and footer bytes. The JPEG format is one of the few formats that has 2 bytes that tell you it is the end of the file.

The header bytes are `FF D8` and the footer bytes are `FF D9`, so if we open the file in a hex editor, we can look for these.

```
$ xxd garden.jpg |less
```

In the first couple of bytes in the first line we see `FF D8`.

```
00000000: ffd8 ffe0 0010 4a46 4946 0001 0101 0048  ......JFIF.....H
```

And if we go all the way to the bottom, the last 2 bytes aren't `FF D9`.
This means it is likely there is extra information at the end of the file.

```
$ xxd garden.jpg |tail
00230500: d9f9 9f63 4b2b c1e5 daf2 7b59 db49 4ba3  ...cK+....{Y.IK.
00230510: f43e b881 5e30 1060 8030 47d6 bacb 58cb  .>..^0.`.0G...X.
00230520: 1046 07b5 7216 df7e 5ff7 c576 363f ebab  .F..r..~_..v6?..
00230530: b70d 18ce 3ccd 6a7e 6b8d af56 b579 39ca  ....<.j~k..V.y9.
00230540: eeef 53ae 8620 31b8 751f 9514 f7fb cff5  ..S.. 1.u.......
00230550: a2bb bdac 9687 98e4 d3b2 e87f ffd9 4865  ..............He
00230560: 7265 2069 7320 6120 666c 6167 2022 7069  re is a flag "pi
00230570: 636f 4354 467b 6d6f 7265 5f74 6861 6e5f  coCTF{more_than_
00230580: 6d33 3374 735f 7468 655f 3379 3363 4438  m33ts_the_3y3cD8
00230590: 6241 3936 437d 220a                      bA96C}".
```

The extra data contains our flag for the challenge with a little message following `FF D9`.

`picoCTF{more_than_m33ts_the_3y3cD8bA96C}`.

We could have also solved this challenge by running strings on the file.

```
$ strings garden.jpg |tail
h--@3
cZi-
M(.I
]hWP&
jc#k
=7g&
mjx/
s\]|."Ue
\qZf
Here is a flag "picoCTF{more_than_m33ts_the_3y3cD8bA96C}"
```
