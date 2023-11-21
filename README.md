# bnd-merch-gewinnspiel-loesung
Eine Anleitung zur Lösung der BND Merch Challenge (November 2023)

**Challenge**

Die Challenge wird in diesem Video von Florian Dalwigk beworben und genauer erklärt.

https://www.youtube.com/watch?v=hjTRV4Fz0DI

Das Repo dazu: https://github.com/bndchallenge/takur_medicine

**Lösung**

**Schritt 1 - Zip entschlüsseln**

Hier wurde die Zip Datei mit diesem Programm verschlüsselt:
```python
import random

def encrypt(file_path, output_path):
  with open(file_path, 'rb') as file:
    m = file.read()
  k = bytearray(random.getrandbits(8) for _ in range(4))
  s = 4
  c = bytearray()
  for i in range(0, len(m), s):
    b = m[i:i+s]
    c.extend(bytearray(b[i] ^ k[i % 4] for i in range(len(b))))

  with open(output_path, 'wb') as file:
    file.write(c)

if __name__ == "__main__":
  input_file = 'geheim.zip'
  encrypted_file = 'geheim.zip.crypt'
  encrypt(input_file, encrypted_file)
```

Zusammengefasst liest das Programm die Zip Datei, generiert einen zufälligen 32 Bit Array bzw. 4 bytes und verschlüsselt den Dateiinhalt mit der XOR Cipher bei der der Array immer wieder alle 4 Bytes verwendet wird. Der verschlüsselte Inhalt wird dann in der .crypt Datei gespeichert.

Wir wissen aber, dass das Zip format natürlich gewisse Eigenschaften hat und so kommt es, dass Zip Dateien üblicherweise mit den 4 Bytes `PK\x03\x04` beginnen. So brauchen wir nur für die jeweils ersten 4 Bytes der .crypt Datei alle 256 Kombinationen mit der XOR Cipher bruteforcen und wir haben den Key.

```python
f = open('geheim.zip.crypt', 'rb')
content = f.read()
zipFormat = b'PK\x03\x04'
key = []
for charIndex in range(4):
    for i in range(256):
        current = bytearray([i])[0]
        decByte = (content[charIndex] ^ current)
        if decByte == zipFormat[charIndex]:
            key.append(current)
            break
print(bytes(key))
```
Output: `b'!Rv\xbc'`

Mit diesem Key können wir dann einfach mit XOR entschlüsseln.
```python
key = b'!Rv\xbc'
c = bytearray()
s = 4
for i in range(0, len(content), s):
    b = content[i:i+s]
    c.extend(bytearray(b[i] ^ key[i % 4] for i in range(len(b))))
with open("decrypted.zip", 'wb') as file:
    file.write(c)
```
So erhalten wir nun die entschlüsselte `decrypted.zip`. Diese ist nun mit einem Passwort geschützt, welches geknackt werden muss.

**Schritt 2 - Zip Passwort knacken**

Die Zip ist sehr einfach zu entschlüsseln. Es muss lediglich mit hashcat der hash der Zip Datei mit der klassischen *rockyou.txt* wordlist abgeglichen werden. Wie man einfach mit Hashcat auf Zip Dateien Wordlistattacken durchführt ist in vielen Tutorials beschrieben.

Wir erhalten das Passwort:
`roneisha2209`

Geben wir das Passwort für die Zip Datei ein, erhalten wir die Dateien `verwahrgelass.kdbx` und `KeePass.DMP`.


**Schritt 3 - KeePass Passwort knacken**

Auch hier muss ganz einfach die Keepass Datei gebruteforced werden, wie es in vielen Tutorials beschrieben ist. Das Passwort befindet sich erneut auf der *rockyou.txt*.

Wir erhalten das Passwort:
`spongebob123`

(Die `KeePass.DMP` ist irrelevant und soll wahrscheinlich auf eine falsche Fährte führen :) )

**Die KeePass Datei**

In der Datei bedindet sich der Link: `https://bnd.bund.de/w4rme-s0cken-4-dec0der`

Mit dem Lösungswort: `BND{M3rcH=c001!}`

Zudem die Nachricht:

﻿`Herzlichen Glückwunsch! Nutze das Losungswort (siehe Password-Feld) auf der steng geheimen Webseite https://bnd.bund.de/w4rme-s0cken-4-dec0der und gewinne mit etwas Glück BND-Merch :)`
