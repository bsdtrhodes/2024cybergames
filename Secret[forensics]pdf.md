The Secret [Forensics] challenge had me kicking myself. The first thing I did was run some commands on this file, including:

binwalk
strings
stegohide

Nothing. So, I opened the file and felt I would have to do something to remove this black line. Turns out, you just need
to copy and past the bugger:

SIVBGR{C0nta1n_Th3_Al13ns}

FYI: Now, this challenge may be harder for different environments. I am using Thunar to look at the directory
and the Atril document viewer to see the PDF file. I kicked myself because I should have checked this first but in fairness,
Firefox opened instead of downloading it, and I never inspected the redacted areas first.
