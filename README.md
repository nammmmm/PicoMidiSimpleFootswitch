# PicoMidiSimpleFootswitch
A simple helper for beginners like me on How to create a simple Footswitch Midi Controller to interact with DAW effects using AdaFruit CircuitPython and a Raspberry Pi Pico

------------------------------------------------------------------------

Ce pedalboard est ultra simpliste. Pas de LED, pas de fioriture. Il envoie des commandes MIDI à l'ordinateur quand on appuie sur les boutons, c'est tout.

Quand j'ai voulu faire ce projet, j'ai du lire un paquet de documents, de tutos, de projets.
Pour la simple raison que tout le monde fait des trucs compliqués.
Je n'ai jamais réussi à trouver un code exemple simple pour débuter, donc le voici. --> Quand on appuie sur un bouton, ça envoie une commande midi, basta.

(info : DAW = Digital Audio Workstation (ou STAN) quand j'utilise ce terme, je fait plutot référence à Waveform, Ableton, Reaper etc)

## Requis logiciel :

- Le firmware AdaFruit Circuit Python qui facilite le MIDI grandement  ([https://circuitpython.org/board/raspberry_pi_pico/](https://circuitpython.org/board/raspberry_pi_pico/))  
Téléchrgez le fichier .UF2  
![dwnuf2](https://user-images.githubusercontent.com/20555863/229300488-b27a0ce0-ad1f-4fe7-ad3f-ece6d0f1adfa.PNG)

- les librairies nécessaires ([https://circuitpython.org/libraries](https://circuitpython.org/libraries))  
![bundle](https://user-images.githubusercontent.com/20555863/229300518-d561407f-e027-4b22-b940-78369d0add78.PNG)



## Requis Matériel :

- Un cable USB adapté (mon Pico est en micro-USB par exemple)  
  (Attention de bien avoir un cable qui supporte le transfert de données/fichiers et pas seulement la charge, oui oui, ça existe et je me suis déjà fait avoir)
  
- Des Switchs momentanés normalement OFF  
  contact temporaire quand on appuie. "Soft Momentary switch OFF(ON) SPST (Simple Pole Simple Throw)    
  Aussi noté normally open au lieu de OFF(ON).    
  Pour ce pedalboard, j'ai préféré des formats courts (longueur du col) cela dépend de la matière à traverser.    
  Le plus simple pour moi a été d'aller sur musikding.de et de prendre ["1PST momentary footswitch soft short"](https://www.musikding.de/1PST-momentary-footswitch-soft-short)    
  C'est en Europe donc pas de douane, et c'est rapide.
  
  
![1pst-momentary-footswitch-soft-short](https://user-images.githubusercontent.com/20555863/229297094-54140d28-9d3f-473d-b734-d60d686003ff.jpg)

- Raspberry Pi Pico

![pico](https://user-images.githubusercontent.com/20555863/229302332-bb475d3b-1965-4ebc-8a4b-6293a981c5c3.png)
![pico2](https://user-images.githubusercontent.com/20555863/229302269-ea9d72c3-1d6f-4e7b-9a02-fc843d981203.png)

---------------------------------------------

# Le process

* Flasher le firmware AdaFruit sur le Raspberry
* Ajouter les librairies necessaires sur le Raspberry
* Ajouter le code adéquat sur le Raspberry
* Faire les branchements électroniques

## Flasher le Firmware AdaFruit sur le Raspberry Pi Pico
Rien de plus simple :
#### Vous avez en votre possession les fichiers .U2F et les librairies téléchargés
![A télécharger](https://user-images.githubusercontent.com/20555863/229302928-6e542c13-350a-4a4a-ad4d-e81bee47c365.png)


- Lors du branchement du Pico par USB à l'ordinateur, restez appuyé sur le bouton __"Bootsel"__ et branchez le cable usb.  
![picobootsel](https://user-images.githubusercontent.com/20555863/229302461-7b08c8c3-72af-4bf1-890d-a7ac7ea78923.png)
- Le Pico devrait être reconnu comme stockage USB (vide).
- Copier/Coller le fichier .U2F précédemment téléchargé sur le stockage du Pico.
- Voila, c'est flashé. Rien de plus!


## Ajouter les librairies necessaires sur le Raspberry Pi Pico
Le Pico devrait avoir redémarré avec à l'intérieur plusieurs fichiers et dossiers

- Dans les fichiers Bundles téléchargés précédemment contenant les librairies
  - vous trouverez un dossier __"lib"__
- Copier/Coller depuis le dossier __"lib"__ téléchargé vers le dossier __"lib"__ du Pico les dossiers et fichiers suivant :  
![librairies à inclure](https://user-images.githubusercontent.com/20555863/229303356-b9b157ab-4eed-4f7f-b715-8219c9513126.png)


## Ajouter le code adéquat sur le Raspberry Pi Pico
Dernière manipulation de fichier :  
Le Raspberry Pi Pico utilise la programmation Python. Pour donner des ordres au Pico, il existe un fichier __"code.py"__ sur le Pico.  
Ce fichier contient les informations programmtiques de votre pédalboard.
Je partage ici mon code fonctionnel utilisé pour mon pedalboard. Ce code est très commenté, il y à trois parties que vous devrez modifier à votre convenance, mais sans connaissance en programmation vous pourrez le faire en suivant les instructions.
- Ouvrez le fichier __"code.py"__ avec tout simplement le bloc note 
- Copier/coller le code suivant dans votre fichier __"code.py"__ et sauvegardez
```python
import time
import board
import usb_midi
import adafruit_midi
import adafruit_ticks
from adafruit_debouncer import Debouncer
from digitalio import DigitalInOut, Direction, Pull
from adafruit_midi.control_change import ControlChange

#  trois parties du code à modifier par l'utilisateur
#  selon ses besoins marqués par [UTILISATEUR]


#  [UTILISATEUR]
#  Configuration du canal midi en base 0
#  Si un seul équipement midi branché mettre 1
midi_channel = 1
midi = adafruit_midi.MIDI(midi_out=usb_midi.ports[1],
                          out_channel=midi_channel-1)

#  [UTILISATEUR]
#  Chaque branchement utilisé sur le Raspberry pico pour les boutons
all_pins = (board.GP2, board.GP3, board.GP4, board.GP5, board.GP6,
             board.GP7, board.GP8, board.GP9, board.GP10, board.GP11,
             board.GP12, board.GP13)

#  [UTILISATEUR]
#  Liste des commandes envoyées à l'activation des boutons
#  A définir soit même arbitrairement
#  Dans le même ordre que les listes des bouttons/pins
#  ex : 20 sera le ControlChange pour le boutton branché sur GP2
midi_events = [20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31]


#  Implémentation automatique de la liste des boutons
#  pour chaque pins utilisé plus haut
#  Avec debouncer pour éviter les "double clic"
buttons = []
for pin in all_pins:
    tmp_pin = DigitalInOut(pin)
    tmp_pin.direction = Direction.INPUT
    tmp_pin.pull = Pull.UP
    buttons.append(Debouncer(tmp_pin))

#  Définition de la liste du suivi d'états
midi_val = [False, False, False, False, False,
             False, False, False, False, False,
             False, False]


#  Boucle de lecture des changements d'état des boutons
while True:
    for i in range(len(buttons)):
        buttons[i].update()
        button = buttons[i]
        #  Si le bouton est appuyé...
        if button.fell:
            #  pressed
            max = 0
            #  Si l'ancienne valeur est "aucune" ou "activé" = envoi de la valeur "0" "désactiver l'effet"
            #  Si l'ancienne valeur est "désactivé" = envoi de la valeur "127" "activer l'effet"
             if midi_val[i] is True:
                max = 127
                midi_val[i] = False
            else:
                midi_val[i] = True
            #  signal midi envoyé
            midi.send(ControlChange(midi_events[i], max))
        #elif button.rose:
            #  released



#  Ce code simpliste ne permet pas de communiquer avec le DAW
#  De ce fait, le programme n'a aucune idée de l'état réel de vos effets.
#  Au lancement, il faut donc activer désactiver chacun des effets souhaités
#  avec le pedalboard pour que les états réels et le programme soient en phase
#  exemple : la disto est désactivée au lancement du DAW
#   le pédalier enverra pourtant la valeur 0 au premier appui pour la désactiver quand même,
#   rien ne se passera donc.
#   le programme sera ensuite bien en état "désactivée" pour la disto à partir de là (en phase)
#   toutes les commandes suivantes fonctionneront correctement
```

#### Le code est modifiable par bloc note à tout moment directement sur le Pico. Quand le code est changé et le fichier sauvegardé, le Pico redémarre pour le prendre en compte directement.

## Faire les branchements électroniques
Côté Hardware, Nous allons nous intérresser uniquement aux Pins __"GND"__ et __"GPxx"__ sur le Pico  
![picoGround](https://user-images.githubusercontent.com/20555863/229304871-c78b4f67-71de-4415-b7f5-c36a92f85d53.png)
![picoGP](https://user-images.githubusercontent.com/20555863/229304892-a6f3aeb5-b84d-4a9a-be52-c41c00b4670a.PNG)

Les bornes __"GND"__ sont toutes reliés entre-elles, donc qu'importe celle que vous choisirez pour des questions pratiques, c'est OK.  
Il est possible de brancher chaque bouton sur une borne __"GND"__, mais aucun intérêt, donc pour la facilité, j'ai relié une patte de chaque Bouton entre-elles, et au final, celà me fait une masse à brancher pour tous les boutons en une seule soudure. (GND signifie Ground : c'est juste la masse)

Ensuite il suffit de brancher l'autre patte de chaque bouton sur une des Bornes __"GPxx"__, l'ordre n'a aucune importance. Vous pouvez très bien utiliser __"GP2"__ / __"GP6"__ / __"GP11"__ / __"GP22"__ pour 4 boutons. ou simplement __"GP2"__ / __"GP3"__ / __"GP4"__ / __"GP5"__. Question de pratique niveau place seulement.

Voici un schéma qui démontre mes grandes capacités Paint, exemple d'un pédalboard avec 4 Boutons seulement.  
![electro](https://user-images.githubusercontent.com/20555863/229305422-21408efd-cbc0-42da-9d0a-02c7eb25f3ec.png)

#### Question électronique, voilà, c'est fait. Vraiment. Pas besoin d'alimentation car l'usb s'en chargera. C'est vraiment tout.




# Quelques précisions sur le Code Python du fichier __"code.py"__
### La première partie de code à modifier par l'utilisateur :  
`midi_channel = 1`  
Le chiffre __"1"__ correspond au premier équipement MIDI sur votre ordinateur. Si vous en avez plusieurs, à vous de trouvez le bon canal.

### La deuxième partie de code à modifier par l'utilisateur :  
```python
all_pins = (board.GP2, board.GP3, board.GP4, board.GP5, board.GP6,
             board.GP7, board.GP8, board.GP9, board.GP10, board.GP11,
             board.GP12, board.GP13)
```
Dans ce code, j'ai 12 boutons.  
Le code modifié qui correspondrait à notre exemple de 4 Boutons serait le suivant :  
`all_pins = (board.GP2, board.GP3, board.GP4, board.GP5)`  
Vous noterez bien que les chiffres utilisés sont ceux sur lesquels ont a branché les Boutons sur le Pico.

### La dernière partie de code à modifier :  
`midi_events = [20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31]`  
C'est la liste des identifiants du signal ControlChange Midi envoyé à votre ordinateur. C'est comme celà que votre DAW sait quel Bouton a été activé.  
Chacun des nombres correspond au __"GPxx"__ de la première liste `all_pins` dans l'ordre donné.  
C'est à dire, dans notre exemple à 4 Boutons le code sera modifié comme suit :  
`midi_events = [20, 21, 22, 23]`  
Alors __GP2__ correspondra à __20__, __GP3__ à __21__, __GP4__ à __22__ et __GP5__ à __23__. Tout simplement.  
Le choix des nombres est arbitraire dans la limite de 0 à 127. Certains codes sont réservés par les DAW parfois, donc personnellement, j'utilise les codes à partir de 20.  
Maintenant si vous voulez plus de détails sur la norme en général [https://fr.audiofanzine.com/mao/editorial/dossiers/le-midi-les-midi-control-change.html](https://fr.audiofanzine.com/mao/editorial/dossiers/le-midi-les-midi-control-change.html)


Voilà c'est tout. Ne reste plus qu'a mettre ça dans un boitier et à configurer votre logiciel de MAO en attribuant les boutons aux effets souhaités.

Pour info, c'est mon premier test avec python/raspberry/midi. Donc ce n'est peut-être pas empiriquement bien fait, mais ça fonctionne vraiment top chez moi et ça a le mérite d'être simple.
