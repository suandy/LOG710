# LOG710
## Cours 1
### Système d'exploitation
Un logiciel qui agit comme intermédiaire entre l'utilisateur et le matériel d'un ordinateur.
### Objectifs d'un OS
1. Fournir un environnement d'exécution pour les programmes.
2. Rendre un système plus facile à utiliser.
3. Utiliser le matériel de l'ordinateur plus efficacement.
### Mécanisme des interruptions
* L'OS est responsable de la gestion du matériel composant le système informatique.
* Le CPU doit communiquer avec des périphériques externes.
    * asynchrones (ayant des cycles d'horloges différents)
    * beacoup moins rapide
* Comment le CPU peut être capable de traiter les évènements générés par les périphériques externes et répondre 
rapidement?
* Comment le CPU peut faire du travail utile en attendant ces évènement?
#### Scrutement (polling)
|Fonctionnement|Avantage|Inconvénients|
|---|---|---|
|Aller vérifier l'état des périphériques (s'ils ont des requêtes?, terminé?) de temps en temps|Simple|1. Prends du temps de la CPU même s'il n'y a pas de requêtes|
| | |2. Quand faire le polling? Difficile à prédire.|
#### Interruption
|Fonctionnement|
|---|
|Chaque périphérique peut envoyer un signal à la CPU pour indiquer un évènement (requête)|
|Pas de coûts d'interrogation des périphériques lorsqu'il n'y a pas de requêtes|
|Réponse plus rapide|
## Cours 2
### Exercice
#### Exercice 01
On considère le code Listing 1. Y compris le processus parent initial, déterminer le nombre de processus qui sont créés par le programme.
```c
#include <stdio.h>
#include <unistd.h>
int main() {
    /* fork a child process */
    fork();
    fork();
    fork();
    return -;
}   /* Listing 1 */
```
##### Réponse 
8 processus
#### Excercice 02
On considère le programme Listing 2. Quelle est la sortie du programme au niveau des lignes A, B, C et D. On suppose que le pid du processus parent est 2600 et celui du fils est 2603.
```c
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>
int main() {
    pid_t pid, pid1;
    
    /* fork a child process */
    pid = fork();
    
    if(pid < 0){
        fprintf(stderr, "Fork failed");
        return 1;
    }
    else if (pid == 0){
        pid1 = getpid();
        printf("pid = %d", pid); /* A */
        printf("pid1 = %d", pid1); /* B */
    }
    else{
        pid1 = getpid();
        printf("pid = %d", pid); /* C */
        printf("pid1 = %d", pid1); /* D */
        wait(NULL);
    }
    return 0;
}   /* Listing 2 */
```
##### Réponse
Ligne /* A * / --> 0   
Ligne /* B * / --> 2603   
Ligne /* C * / --> 2603   
Ligne /* D * / --> 2600
#### Exercice 03
On considère le programme Listing 3. Expliquer la sortie au niveau de la ligne A.
```c
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int value = 5;

int main() {
    pid_t pid, pid1;
    
    /* fork a child process */
    pid = fork();
    
    if (pid == 0){
        value += 15;
        return 0;
    }
    else if(pid > 0){
        wait(NULL);
        printf("value = %d", value); /* A */
        return 0
    }
}   /* Listing 2 */
```
##### Réponse
À la ligne A,, la variable value est toujours égale à 5 (le processus fils a sa propre copie de la variable value donc 
l'incrémentation value += 15 se fait sur la copie du fils et n'a aucun effet sur celle du processus père)

#### Exercice 04
On considère l'instruction suivante:
```c
for(int i=0; i<3; i++)
    fork();
```
On suppose que l'appel système `fork()` ne retourne pas d'erreurs.   
a) Quel est le nombre de processus créés par l'instruction suivante?   
b) Donnez l'arborescence des processus.
##### Réponse
a) 08 processus   
b) ![exercice 4b](images/cours2/exercice4.png)
## Cours 3
## Cours 4