### Catégorie :
Stéganographie

### Flag : 
Hero{0NL1NE_700L_0V3RR473D}

### Auteur du challenge :
Thibz

### Auteur du WU :
moi(ScottKushy)

### Desc : 
Don't try to throw random tools at this poor image. Go back to the basics and learn the detection techniques.

A little hint to make your life easier : 200x200

Good luck!

### Write Up:

Dans ce challenge, la description et le titre indique déja pas mal ce qu'il faut faire, "LSD" peut faire facilement penser à l'une des méthodes en stéganographie la plus connue : le LSB (Last Significant Byte).
La description est elle aussi un gros lapsus révélateur mais j'expliquerai pourquoi elle est importante par la suite.

Avant toutes choses au niveau de la reconnaissance on essaye de lancer les classiques de la stéganographie : StegSolve, StegoVeritas, Binwalk, ExifTool, Zsteg, DiE(ça c'est plus personnel, c'est un peu ma madelaine de proust).

En essayant tout ces outils, on s'aperçoit une chose qui reviens le plus souvent : un bout de phrase bizarre expliquant la définition de ce qu'est la stéganographie. Via le green pattern
J'ai donc décidé de lancer StegSolve pour voir ce qu'il en était de ce fameux pattern on next le pattern on next le pattern et OH! une fois arrivé au green pattern, au bit 0 on aperçoit ceci : 

![image_2023-05-17_180819853](https://github.com/xTommyBoy/wu/assets/66128183/39f60dea-953d-4132-a179-3f93e00449f5)

C'est donc bien du LSB !

Je décide donc d'extraire ce fameux LSB via StegSolve : 

![image_2023-05-17_180907006](https://github.com/xTommyBoy/wu/assets/66128183/a3c9eae9-f78c-43bf-b9e1-62c9b98ae05c)

Mais...Bizarrement on obtient que des bouts de phrases de cette fameuse définition, de ce qu'est la stéganographie. Je décide donc de sauvegarder l'extraction et de regarder les strings via DiE, et en tentant d'écrire "Her" dans le filtre voilà ce qu'on obtient :

![image_2023-05-17_180931266](https://github.com/xTommyBoy/wu/assets/66128183/d98515ff-913b-451d-8ba2-225e616c6740)

à la toute fin du fichier, on obtient la phrase "Here is your flag" coupé et sans suite....

![image_2023-05-17_180946882](https://github.com/xTommyBoy/wu/assets/66128183/e19a72f9-0594-4ef2-9007-981753a78c3e)

> **_NOTE:_** En essayant d'extraire le LSB par CyberChef ou autre site capable de faire ça, et bien il ne reconnaissait même pas le LSB en question...

C'est donc à partir de la que j'ai décidé d'investiguer l'image de manière plus profonde.

En premier lieu, j'ai tenté une floppée de techniques et d'outils le plus profondément possible mais en vain.

J'ai donc décidé de voir pourquoi est ce qu'on ne pouvait identifier le lsb que par la couleur verte. J'ai donc utilisé un Pixel Color Identifier pour voir le pourquoi du comment :

![image_2023-05-17_180958907](https://github.com/xTommyBoy/wu/assets/66128183/7613a47a-ede9-4832-bbd7-24f1d4c51c1a)

Et voilà tout est la ! La technique se repose en fait sur deux choses : les pixels cachés sont en jaune car les pixels sont reconnaissables que en vert et qu'aucun layer ou pattern bleu ne fonctionne (donc 0 RGB value). 
Le rouge lui est fortement identifiable. Si on enregistre l'image avec le LSB visible dans la partie en haut à gauche (la ou est le LSB en question) il reste constamment en 255 d'ou sa forte présence, le vert lui est présent mais ces valeurs sont très changeantes : 254,255,254,255 etc... ce qui nous indique donc que le LSB ce base sur une différence de valeur entre le rouge et le vert pour former le LSB.

> **_NOTE:_** Le changement de valeurs de 255 à 254 est évidemment inperceptible à l'oeil nu c'est la tout l'intérêt du concept.

Maintenant deux choix s'offre à nous, maintenant que nous savons que tout ce base sur cet écart de pixels nous pouvons l'obtenir de deux façons : soit en écrivant un script avec PIL : 

```python
def stringToFile(string, dst):
    file = open(dst, 'w')
    file.write(string)
    file.close()

def binaryToString(binary):
    return ''.join([chr(int(binary[i:i+8], 2)) for i in range(0, len(binary), 8)])

def decode(src):
    print("[-] Decoding... ")
    
    extracted_bin = ""
    img = Image.open(src, 'r')

    width, height = img.size
    print("[-] Image size: {}x{}".format(width, height))
    for x in range(0, 200):
        for y in range(0, 200):
            pixel = list(img.getpixel((x, y)))
            # Pixel[1] is the green plane
            extracted_bin = extracted_bin + str((pixel[1] & 1))
    
    stringToFile(binaryToString(extracted_bin), "extracted.txt")
    print("[-] Message extracted !")
    print("[-] Message saved as extracted.txt")

``` 
(J'ai pris l'éxemple du WU de HeroCTF), soit utiliser la deuxième solution : l'extraire nous même ! (ce que j'ai fait).

On récupère donc notre image avec le lsb visible (green pattern 0 bits), on screenshot la partie lsb en haut à gauche en 200 par 200 (comme expliqué dans la description pour être sur de tout récuperer),
Et ensuite on extrait le tout : 

![image_2023-05-17_181014507](https://github.com/xTommyBoy/wu/assets/66128183/18086e7b-3409-497f-842c-b05e1363322a)

Ce qui nous affiche le reste du texte !
(Dans cet exemple j'ai utilisé CyberChef, mais tout autre tool d'extraction fonctionnerait aussi.)

Ce qui nous donne le flag : Hero{0NL1NE_700L_0V3RR473D}
