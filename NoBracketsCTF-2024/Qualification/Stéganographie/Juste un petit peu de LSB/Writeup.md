# Write Up - Juste un petit peu de LSB

## Description

Challenge de stéganographie visant à retrouver un message caché dans un fichier .wav fourni. Message sous la forme `NBCTF{...}`

[Juste un petit peu de LSB](challJLSB.png)

## Solution

### 1ère Étape : Type de stéganographie sur fichier WAV

Tout d'abord, les fichiers WAV sont des fichiers stockant des échantillons sonores sans compression avec perte.

Cela permet de pouvoir lire les échantillons et modifier leur LSB (Bit de poids faible) dans le but de cacher des informations.

Ce qui est le cas ici.

### 2ème étape : Recherche de module permettant de lire / écrire les échantillons audio.

Il existe en Python une module permettant cela, le `wav` module.

Ce module va nous permettre par la suite de manipuler le fichier `chall.wav` afin de retrouver le message.

Après avoir réalisé des recherches ont tombent sur un site internet expliquant, à travers le module `wav` l'analyse d'échantillons audio : [Stéganographie LSB]("https://daniellerch.me/stego/intro/lsb-en/#lsb-steganography-in-wav-audio-files")

### 3ème Étape : Découverte du module

Le module va nous permettre de lire les échantillons :

```python
cover_wav = wave.open("chall.wav", mode='rb')
frames = bytearray(cover_wav.readframes(cover.getnframes()))
```

Grâce au script précédent, on peut maintenant modifier les échantillons en ajoutant une unité par exemple :

```python
frames[0] += 1

stego_wav = with wave.open('stego-sound.wav', 'wb')
stego_wav.setparams(cover_wav.getparams())
stego_wav.writeframes(bytes(frames))
```

### 4ème Étape : Exctraction du message caché :

À l'aide des exemples vu précédemment, on peut former un script permettant de récupérer un message caché dans le LSB.

Cependant, le format WAV stocke généralement les échantillons avec une précision de 16bits. Or, nous souhaitons uniquement modifier l'octet qui représente les bits les moins significatifs : LSB.

On importe le module, on lit les frames et on initialise les variables nécessaires.

**Importation et définition**

```python
import wave

cover_wav = wave.open("chall.wav", mode='rb')

frames = bytearray(cover_wav.readframes(cover_wav.getnframes()))

message_ex = []
value = 0
j = 0
```

**Récupération des bits cachés**

Ensuite, on réalise une boucle qui va parcourir les données audio pour extraire les bits cachés.

```python
for i in range(0, len(frames), 3):
    msg_bit = frames[i] % 2
    if j % 8 == 0 and j != 8:
        message_ex.append(value)
        value = 0
    value |= msg_bit << (j % 8)
    j += 1
```

Ici, les échantillons audios sont stockés en paquets de 3 octets (24 bits) ce qui explique un **pas de 3** dans la boucle.

Puis, à chaque réitération de la boucle on doit vérifier que notre compteur de bits `j` n'est pas égal à 8 car nous récupéront 8 bits (1 octet) pour un caractère formé.

Ensuite, on extrait le LSB de chaque octet avec le modulo (reste) de la division par 2.

Puis, à chaque octet collectés, un caractère est formé, carrècte que l'on ajoute à la liste `message_ex`.

**Conversion vers caractère**

À la fin de notre boucle, notre liste contient tout les bits cachés.

On va, à l'aide des caractères ASCII, retranscrire nos bits en caractère.

```python
r = ''.join([chr(l) for l in message_ex])
```

On créer une variable contenant tout les bits cachés sous forme de chaîne de texte.

Pour terminer, on créer un fichier `extracted_message.txt` et on lui écrit tous les caractères décodées :

```python
with open("extracted_message.txt"), "w", encoding='utf-8') as file:
    file.write(r)
```

### 5ème étape : Script final

L'analyse précédente nous permet de former le script solution du challenge.

On a :

```python
import wave

cover_wav = wave.open("chall.wav", mode='rb')

frames = bytearray(cover_wav.readframes(cover_wav.getnframes()))

message_ex = []
value = 0
j = 0

for i in range(0, len(frames), 3):
    msg_bit = frames[i] % 2
    if j % 8 == 0 and j != 8:
        message_ex.append(value)
        value = 0
    value |= msg_bit << (j % 8)
    j += 1

r = ''.join([chr(l) for l in message_ex])

with open("extracted_message.txt", "w", encoding='utf-8') as file:
    file.write(r)
```

Après éxecution on obtient un fichier `extracted_message.txt` contenant une quantité monstre de caractère.

Il nous ne reste plus qu'à faire un `CTRL F` et rechercher le format de texte demandé, en l'occurrence : `NBCTF`

On tombe sur : `=5Ó>NBCTF{D0_nOt_m3lt_th3_L5B}      CREDITS : VINCE HAWK & NAMINÉ - RIGHT HERE (Fm)`

On conclus que le flag de ce challenge est :

`NBCTF{D0_nOt_m3lt_th3_L5B}`

## Author

By Sim - NoBrackets CTF 2024
