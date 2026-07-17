# Author: cvpbPGS{arfgrq_rap0qvat_s3n24s73}

# Description
We intercepted a suspiciously encoded message, but it’s clearly hiding a flag. No encryption, just multiple layers of obfuscation. Can you peel back the layers and reveal the truth?

Download the message (NjM3NjcwNjI1MDQ3NTMyNTM3NDI2MTcyNjY2NzcyNzE1ZjcyNjE3MDMwNzE3NjYxNzQ1ZjczMzM2ZTMyMzQ3MzM3MzMyNTM3NDQ=).

# Hints
1. The flag has been wrapped in several layers of common encodings such as ROT13, URL encoding, Hex, and Base64. Can you figure out the order to peel them back?
2. A tool like CyberChef can be interesting.

# Steps
1. Fire up CyberChef.
2. Select From Base64 to Recipe section because the first layer has = at the end.
3. Next layer (637670625047532537426172666772715f72617030717661745f73336e3234733733253744) looks like Hex because it consists of number and some character. So we select From Hex.
4. Next layer (cvpbPGS%7Barfgrq_rap0qvat_s3n24s73%7D) has the %7B and %7D which might be URL encoding so we try URL decode.
5. Last layer (cvpbPGS{arfgrq_rap0qvat_s3n24s73}) looks like a normal ROT13.

Answer: picoCTF{nested_enc0ding_f3a24f73}
