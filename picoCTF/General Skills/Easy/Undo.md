# Author: Yahaya Meddy

# Description
Can you reverse a series of Linux text transformations to recover the original flag? Additional details will be available after launching your challenge instance.

# Hints
1.  For text translation and character replacement, see tr command documentation.

# Steps
1. Use the given nc command to connect to the remote server.
2. Follow the instruction of each steps. Put the right command to pass through.
```
===Welcome to the Text Transformations Challenge!===

Your goal: step by step, recover the original flag.
At each step, you'll see the transformed flag and a hint.
Enter the correct Linux command to reverse the last transformation.

--- Step 1 ---
Current flag: KXM5MzA0MG5zLWZhMDFnQHplMHNmYTRlRy1nazNnLXRhMWZlcmlyRShTR1BicHZj
Hint: Base64 encoded the string.
Enter the Linux command to reverse it: base64 -d
Correct!

--- Step 2 ---
Current flag: )s93040ns-fa01g@ze0sfa4eG-gk3g-ta1ferirE(SGPbpvc
Hint: Reversed the text.
Enter the Linux command to reverse it: rev
Correct!

--- Step 3 ---
Current flag: cvpbPGS(Eriref1at-g3kg-Ge4afs0ez@g10af-sn04039s)
Hint: Replaced underscores with dashes.
Enter the Linux command to reverse it: tr '-' '_'
Correct!

--- Step 4 ---
Current flag: cvpbPGS(Eriref1at_g3kg_Ge4afs0ez@g10af_sn04039s)
Hint: Replaced curly braces with parentheses.
Enter the Linux command to reverse it: tr '()' '{}'
Correct!

--- Step 5 ---
Current flag: cvpbPGS{Eriref1at_g3kg_Ge4afs0ez@g10af_sn04039s}
Hint: Applied ROT13 to letters.
Enter the Linux command to reverse it: tr 'a-zA-Z' 'n-za-mN-ZA-M' 
Correct!

Congratulations! You've recovered the original flag:
>>> picoCTF{Revers1ng_t3xt_Tr4nsf0rm@t10ns_fa04039f}
```

Answer: picoCTF{Revers1ng_t3xt_Tr4nsf0rm@t10ns_fa04039f}
