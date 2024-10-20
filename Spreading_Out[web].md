Spreading out is a flag that's broken apart and hidden in several areas. Sometimes, these are fun; sometimes, they can be frustrating.
I'd put this in a section of "leans toward frustrating." Like with most of these types of challenges, I instantly look for a
robots.txt and see if it has a flag or a hint as to what file might contain a flag or "piece of flag."

https://uscybercombine-s4-spreading-out.chals.io/robots.txt did have a piece of flag AND hinted how many parts we are
looking for in this challenge. The first one is:

1/5: SIVBGR{ARIA_1s

Moving on, without a recommendation to another file or location, I decided to fuzz this, and ffuf found a .env file that contained:
2/5: _spreading_3v3rywh3r3

ffuf found a /readme file that contained:
3/5: _4lw4ys_4nd

Note that ffuf found all of this information using these two wordlists:
/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt 
/usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-words.txt

With these wordlists, ffuf also found a wwwlog directory, but we have no read permissions. We're going to hit that, too, very soon.
Maybe it's nothing, but it might be helpful too.

I began just trying a large wordlist since sometimes, ffuf wordlists contain different words based on their usefulness. Since we are
allowed (and expected) to fuzz the site, I figured we could kitchen sink this (throw everything including the kitchen sink at this):

ffuf, using /usr/share/seclists/Discovery/Web-Content/CMS/trickest-cms-wordlist/shopware-all-levels.txt, found a sitemap.xml file which contained number 4:

4/5: _c4nnot_b3

Awesome. While ffuf was running, I also tried some other probes because I found an image file on the page and I also wanted to know if maybe
the flag would be in an ETag, cookie, or header somewhere:

wget -S --save-cookies cookie-jar.txt --keep-session-cookies https://uscybercombine-s4-spreading-out.chals.io/

Nothing in session or cookies, and I couldn't download the main image with wget for some reason, BUT Firefox did let me download it,
so that might be a 301 without redirect issue, or did I not copy the link correctly? For the size of the download.png file (396K),
there had to be more to it, so I ran binwalk against it and found something. So, first, try to get the download.png via wget:

┌──(trhodes㉿kali)-[~/Downloads/USC]
└─$ wget -S --save-cookies cookie-jar.txt --keep-session-cookies https://uscybercombine-s4-spreading-out.chals.io/download.png
--2024-06-05 15:57:19--  https://uscybercombine-s4-spreading-out.chals.io/download.png
Resolving uscybercombine-s4-spreading-out.chals.io (uscybercombine-s4-spreading-out.chals.io)... 143.244.222.116, 143.244.222.115
Connecting to uscybercombine-s4-spreading-out.chals.io (uscybercombine-s4-spreading-out.chals.io)|143.244.222.116|:443... connected.
HTTP request sent, awaiting response...
  HTTP/1.1 404 NOT FOUND
  Server: Werkzeug/3.0.3 Python/3.10.14
  Date: Wed, 05 Jun 2024 19:57:24 GMT
  Content-Type: text/html; charset=utf-8
  Content-Length: 207
  Connection: close
2024-06-05 15:57:23 ERROR 404: NOT FOUND.

Since I have it with Firefox, we can ignore wget for now and start working on the image itself:

┌──(trhodes㉿kali)-[~/Downloads/USC]
└─$ du -h download.png
396K    download.png

Why so big? Let's check something:
┌──(trhodes㉿kali)-[~/Downloads/USC]
└─$ binwalk download.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 331 x 711, 8-bit/color RGBA, non-interlaced
54            0x36            Zlib compressed data, compressed

Yeah, okay, let's get this file:

┌──(trhodes㉿kali)-[~/Downloads/USC]
└─$ binwalk -e download.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 331 x 711, 8-bit/color RGBA, non-interlaced
54            0x36            Zlib compressed data, compressed

┌──(trhodes㉿kali)-[~/Downloads/USC]
└─$ cd _download.png.extracted/

┌──(trhodes㉿kali)-[~/Downloads/USC/_download.png.extracted]
└─$ ls
36  36.zlib

┌──(trhodes㉿kali)-[~/Downloads/USC/_download.png.extracted]
└─$ file 36.zlib
36.zlib: zlib compressed data

┌──(trhodes㉿kali)-[~/Downloads/USC/_download.png.extracted]
└─$ zlib-flate -uncompress < 36.zlib > output_file

┌──(trhodes㉿kali)-[~/Downloads/USC/_download.png.extracted]
└─$ file output_file
output_file: data

Looks like a PDF? There is a lot of odd characters here, so, for no other reason than seeing unprintable characters, suggesting
that it's further compressed with something else such as bzip2 or something, or something else is going on here. Binwalk again
and WTF, an embedded file inside of an embedded file:

┌──(trhodes㉿kali)-[~/Downloads/USC/_download.png.extracted]
└─$ binwalk output_file

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
3772          0xEBC           MySQL ISAM index file Version 3
10838         0x2A56          MySQL MISAM compressed data file Version 4
16698         0x413A          MySQL ISAM index file Version 3
17479         0x4447          MySQL ISAM index file Version 3
153289        0x256C9         MySQL MISAM index file Version 3
167059        0x28C93         MySQL MISAM index file Version 3
343140        0x53C64         MySQL ISAM index file Version 5
386283        0x5E4EB         MySQL MISAM compressed data file Version 3
448116        0x6D674         MySQL MISAM index file Version 5
481014        0x756F6         MySQL ISAM index file Version 1
509051        0x7C47B         MySQL ISAM index file Version 7
723354        0xB099A         MySQL MISAM index file Version 7
723622        0xB0AA6         MySQL MISAM index file Version 7
727794        0xB1AF2         MySQL ISAM index file Version 4
788376        0xC0798         MySQL ISAM index file Version 3
850041        0xCF879         MySQL MISAM index file Version 3

Why are MySQL files inside of a zlib compressed file that was ITSELF INSIDE a download.png image? What a rabbit hole. Let's move on, and
annoyingly, binwalk can't extract these:

┌──(trhodes㉿kali)-[~/Downloads/USC/_download.png.extracted]
└─$ binwalk -e output_file

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
3772          0xEBC           MySQL ISAM index file Version 3
10838         0x2A56          MySQL MISAM compressed data file Version 4
16698         0x413A          MySQL ISAM index file Version 3
17479         0x4447          MySQL ISAM index file Version 3
153289        0x256C9         MySQL MISAM index file Version 3
167059        0x28C93         MySQL MISAM index file Version 3
343140        0x53C64         MySQL ISAM index file Version 5
386283        0x5E4EB         MySQL MISAM compressed data file Version 3
448116        0x6D674         MySQL MISAM index file Version 5
481014        0x756F6         MySQL ISAM index file Version 1
509051        0x7C47B         MySQL ISAM index file Version 7
723354        0xB099A         MySQL MISAM index file Version 7
723622        0xB0AA6         MySQL MISAM index file Version 7
727794        0xB1AF2         MySQL ISAM index file Version 4
788376        0xC0798         MySQL ISAM index file Version 3
850041        0xCF879         MySQL MISAM index file Version 3


┌──(trhodes㉿kali)-[~/Downloads/USC/_download.png.extracted]
└─$ ls
36  36.zlib  nextfile  text_file

┌──(trhodes㉿kali)-[~/Downloads/USC/_download.png.extracted]
└─$ ls output_file
nextfile

┌──(trhodes㉿kali)-[~/Downloads/USC/_download.png.extracted]
└─$ ls -al
total 25512
drwxrwxr-x 2 trhodes trhodes     4096 Jun  5 16:52 .
drwxrwxr-x 4 trhodes trhodes     4096 Jun  5 16:19 ..
-rw-rw-r-- 1 trhodes trhodes        0 Jun  5 16:19 36
-rw-rw-r-- 1 trhodes trhodes   402236 Jun  5 16:19 36.zlib
-rw-r--r-- 1 trhodes trhodes   943329 Jun  5 16:22 output_file
-rw-rw-r-- 1 trhodes trhodes 24762600 Jun  5 16:38 text_file

┌──(trhodes㉿kali)-[~/Downloads/USC/_download.png.extracted]
└─$

I took a walk since it seemed I would have to extract these files using the file offset manually, and when I came back,
I noticed in the discord that someone solved this without ever finding the fifth part. So I thought huh, maybe we're just
supposed to complete the sentence. I put parts 1-4 together and noticed that the last word is probably a "l33t" speak
version of stopped. Here is the flag with all four parts:

SIVBGR{ARIA_1s_spreading_3v3rywh3r3_4lw4ys_4nd_c4nnot_b3

So I guessed the last part was: st0pp3d, so let's add that with an underscore and submit this as the flag:

SIVBGR{ARIA_1s_spreading_3v3rywh3r3_4lw4ys_4nd_c4nnot_b3_st0pp3d}

That was it, solved. I still see other people struggling to guess the last part; I think they should try taking a walk as well.
For my own notes, I was going to extract these SQL files like this. I'm only documenting it since it was in my head, and I
think it would be a good reference for anyone reading this solution. The reason I am doing things this way is because binwalk did
not extract the files; there might be an issue with how it's reading the files or something else that maybe dtrace/strace could
pick up on if I had more time to debug:

# First MySQL ISAM index file (0xEBC to 0x2A56)
dd if=nextfile of=output_ebc.isam bs=1 skip=3772 count=7066

# MySQL MISAM compressed data file (0x2A56 to 0x413A)
dd if=nextfile of=output_2a56.misam bs=1 skip=10838 count=5860
