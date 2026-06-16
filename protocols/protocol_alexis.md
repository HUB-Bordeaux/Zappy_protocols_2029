# Protocole broadcast

## Présentation

Le protocole de communication repose sur l'utilisation de **FlatBuffers** pour la sérialisation des données et sur un mécanisme de **chiffrement AES-128 en mode CBC** pour protéger le contenu des messages échangés.

---

# Structure du paquet

Le schéma FlatBuffers utilisé est le suivant :

```fbs
namespace Zappy;

table Broadcast {
    team_signature: string;
    com_type: string;
    target_id: int;
    sender_id: int;
    payload: string;
    quantity: int;
    coords_x: int;
    coords_y: int;
}

root_type Broadcast;
```

## Description des champs

| Champ          | Type   | Description                                    |
| -------------- | ------ | ---------------------------------------------- |
| team_signature | string | Signature ou identifiant de l'équipe émettrice |
| com_type       | string | Type de communication ou action                |
| target_id      | int    | Identifiant du destinataire                    |
| sender_id      | int    | Identifiant de l'émetteur                      |
| payload        | string | Donnée utile transportée                       |
| quantity       | int    | Quantité associée au message                   |
| coords_x       | int    | Coordonnée X                                   |
| coords_y       | int    | Coordonnée Y                                   |

---

# Génération des fichiers FlatBuffers

À partir du fichier de définition :
```bash
Broadcast.fbs
```

Les bindings peuvent être générés pour plusieurs langages via l'outil `flatc`.

Exemple :

```bash
./flatc --cpp --python --ts -o generated Broadcast.fbs
```

Cette commande génère les fichiers nécessaires pour :

* C++
* Python
* TypeScript

Le dossier de sortie sera :

```text
generated/
```

---

# Chiffrement des données

Avant l'envoi du paquet FlatBuffers, le contenu de nos champs seront chiffré à l'aide d'un algorithme AES.

## Algorithme utilisé

Caractéristiques :

* Algorithme : AES
* Mode : CBC (Cipher Block Chaining)
* Taille de clé : 128 bits (16 caractères)
* Padding : PKCS7
* IV : Généré aléatoirement pour chaque message
* Encodage final : Base64

---

# Implémentation Python

```python
#!/usr/bin/env python3

import base64
from Crypto.Cipher import AES
from Crypto import Random
from Crypto.Util.Padding import pad, unpad

class AESCipher:
    @staticmethod
    def encrypt(raw, key):
        raw = raw.encode()
        key = key.encode()
        raw = pad(raw, AES.block_size)

        iv = Random.new().read(AES.block_size)

        cipher = AES.new(key, AES.MODE_CBC, iv)

        return base64.b64encode(iv + cipher.encrypt(raw))

    @staticmethod
    def decrypt(enc, key):
        key = key.encode()

        enc = base64.b64decode(enc)

        iv = enc[:16]

        cipher = AES.new(key, AES.MODE_CBC, iv)

        return unpad(
            cipher.decrypt(enc[16:]),
            AES.block_size
        )

message = "toto"
key = "1234567891011123"

encrypted = AESCipher.encrypt(message, key)
decrypted = AESCipher.decrypt(encrypted, key).decode()

print("Encrypted:", encrypted)
print("Decrypted:", decrypted)

assert message == decrypted
```

# Déchiffrer nos messages

Une fois nos paquets interceptés, FlatBuffers permettra d’en récupérer le contenu. Pour être en mesure de le lire, il suffit d’utiliser notre algorithme de chiffrement avec la bonne clé afin de le rendre lisible. Cette clé est accessible sur notre plateforme `CTF` après avoir réussi plusieurs défis.

# Liens utiles

[Capture The Flag](https://ctf.myepitech.com/)

[FlatBuffers](https://github.com/google/flatbuffers)

[AES Cipher](https://fr.wikipedia.org/wiki/Advanced_Encryption_Standard)
