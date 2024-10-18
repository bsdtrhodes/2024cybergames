This was done post-games. Someone in the chat said they used fcrackzip, which I have never seen before but is available in Kali if you just install it with apt(8). This
was the comment, I think, by a user of "Hugo," and I am posting it completely:
  "oh no! 
  sudo fcrackzip -b -u -v -l 1-7 short_n_sweet.zip 
  That’s how I was able to brute force it. I kept the pw length (-l) between 1-7. 
  Try also using the crack station, and you will get the cleartext immediately."

They are correct; running that line gives the password:

┌──(trhodes㉿kali)-[~/Downloads/unknown]
└─$ fcrackzip -b -u -v -l 1-7 short_n_sweet.zip
found file 'build_different.7z', (size cp/uc    389/   377, flags 9, chk b06c)
found file 'built_different.txt', (size cp/uc    295/   498, flags 9, chk b145)
checking pw B956~

PASSWORD FOUND!!!!: pw == B!p*3

Using the password to unzip the file, we see two new, built_different.7z and built_different.txt:
 
┌──(trhodes㉿kali)-[~/Downloads/unknown]
└─$ unzip short_n_sweet.zip
Archive:  short_n_sweet.zip
[short_n_sweet.zip] build_different.7z password:
 extracting: build_different.7z
  inflating: built_different.txt

┌──(trhodes㉿kali)-[~/Downloads/unknown]
└─$ ls
build_different.7z  built_different.txt  short_n_sweet.zip

There was a follow-up to the comments since the CTF was over, and it noted, as quoted here:
"The built_different.txt file had a bunch of hash values, and one of them spelled out Maramoros, which was the pw for build_different.7z."

I found this is NOT the case, and probably a typo; the password was actually Matamoros." Indeed, Crackstation (<a href="https://crackstation.net/">crackstation.net</a> will
tell you this if you paste the contents of the entire file:

290c5768c4c300bb5d94c5e48566d940	md2	Matamoros
4878e6f0309cb4c0b1653d540fde005e	md4	Matamoros
6c851b965778e56409a4963807c11122	md5	Matamoros
edfa12078ab7c689e2e5a37e30220a737505301ef06bc7f8bdf24c89c7cc7689	Unknown	Not found.
9d5e6dd6e9432865dd6a5ded27682705d2a4ef1a	Unknown	Not found.
55732c74c1d2dadd52c36e2f4ca6031e041ab0ce	sha1	Matamoros
977802f5a40c482b13508874af6f12c3ec2f096db6fc598bbc95754c	sha224	Matamoros
30a09df5dc99880273cfc6dbcd4b48c35e5111082260d23442740ecc21d56e19	sha256	Matamoros
02812275fad36de5b71f1baecd48f685d4f1d64c9f1ed200a981f870cb3a9899
befdb3bb597b394dc97ab9b12566e2a7f4e5587dce267ff02f0f4050fdd6baf8	sha512	Matamoros

Using this, we can extract the final file:
┌──(trhodes㉿kali)-[~/Downloads/unknown]
└─$ 7za e build_different.7z

7-Zip (a) 23.01 (x64) : Copyright (c) 1999-2023 Igor Pavlov : 2023-06-20
 64-bit locale=en_US.UTF-8 Threads:1 OPEN_MAX:1024

Scanning the drive for archives:
1 file, 377 bytes (1 KiB)

Extracting archive: build_different.7z

Enter password (will not be echoed):
--
Path = build_different.7z
Type = 7z
Physical Size = 377
Headers Size = 217
Method = LZMA2:12 7zAES
Solid = -
Blocks = 1

Everything is Ok

Size:       219
Compressed: 377

The new file is: common_passwords_rock.zip and should be in the directory:
┌──(trhodes㉿kali)-[~/Downloads/unknown]
└─$ ls
build_different.7z  built_different.txt  common_passwords_rock.zip  short_n_sweet.zip

It hints that the rockyou wordlist should be used against this zip file. The manual page for fcrackzip states:

       -D, --dictionary
              Select  dictionary  mode.  In  this  mode, fcrackzip will read passwords from a file, which must contain one password per line and should be alphabetically sorted (e.g. using
              sort(1)).
With that, it's time to start cracking. We will use the -D for a dictionary file, but mix it with -u to reduce false positives and -p
so specify the passwords line by line. With this, we have the password and the flag:

┌──(trhodes㉿kali)-[~/Downloads/unknown]
└─$ fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt common_passwords_rock.zip


PASSWORD FOUND!!!!: pw == 123456789123456789

┌──(trhodes㉿kali)-[~/Downloads/unknown]
└─$ unzip common_passwords_rock.zip
Archive:  common_passwords_rock.zip
[common_passwords_rock.zip] flag.txt password:
 extracting: flag.txt

┌──(trhodes㉿kali)-[~/Downloads/unknown]
└─$ cat flag.txt
SIVBGR{P@SSW0RDZ_R_H@RD}

And there's our flag.
