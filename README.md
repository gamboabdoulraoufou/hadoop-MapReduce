## hadoop-MapReduce

In this post, I show how to use **Map/Reduce** Module with streaming API (using Python code) to process data. In this case I make word count.

**1- Create mapper and reducer files**
Create map file called wordcount_mapper.py
Add the code billow

```python
#!/usr/bin/env python   

import sys

for line in sys.stdin:
    line = line.strip()
    keys = line.split() 
    for key in keys:
        value = 1        
        print('{0}\t{1}'.format(key, value))
```

Create reduce file called wordcount_reducer.py
Add the code billow

```python
#!/usr/bin/env python

import sys

last_key      = None            
running_total = 0

for input_line in sys.stdin:
    input_line = input_line.strip()
    this_key, value = input_line.split("\t", 1)
    value = int(value)
 
    if last_key == this_key:
        running_total += value
    else:
        if last_key:
            print( "{0}\t{1}".format(last_key, running_total))
        running_total = value
        last_key = this_key

if last_key == this_key:
    print( "{0}\t{1}".format(last_key, running_total))
```   

Enter the following to make it executable
``` sh
chmod +x wordcount_mapper.py
chmod +x wordcount_reducer.py
```

Cet article est écrit en _mars 2015._  
En ce moment la dernière version de _Spark_ etait **1.3.0**  

**_L'article couvre les points suivants:_**
- Installation des pré-requis
- Installation et construction de spark
- Lancement de spark en mode interactif (Scala et python)
- Deploiement de spark sur un cluster Standalone avec un ou plusieurs noeud
- Création et lancement d'une application Spark

**_Caractérisques:_**
- 2 VM sur Google Compute Engine
- OS: Ubuntu 12.04
- OpenJDK 1.6.0_27
- Scala 2.9.3
- Maven 3.0.4
- Python 2.7.3 
- Git 1.7.9.5 
  
  
### Installation des pré-requis

**Créer un utilusateur sparkmanager**
```sh
sudo adduser sparkmanager # mot de passe: spark
```
