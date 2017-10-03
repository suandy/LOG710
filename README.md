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
|Aller vérifier l'état des périphériques (s'ils ont des requêtes?, terminé?) de temps en temps| Simple | 1. Prends du temps de la CPU même s'il n'y a pas de requêtes |
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
On considère le code Listing 1. Y compris le processus parent initial, déterminer le nombre de processus qui sont créés
par le programme.
```c
#include <stdio.h>
#include <unistd.h>
int main() {
    // fork a child process
    fork();
    fork();
    fork();
    return -;
}   // Listing 1
```
##### Réponse
8 processus
#### Excercice 02
On considère le programme Listing 2. Quelle est la sortie du programme au niveau des lignes A, B, C et D. On suppose que
le pid du processus parent est 2600 et celui du fils est 2603.
```c
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>
int main() {
    pid_t pid, pid1;

    // fork a child process
    pid = fork();

    if(pid < 0){
        fprintf(stderr, "Fork failed");
        return 1;
    }
    else if (pid == 0){
        pid1 = getpid();
        printf("pid = %d", pid); // A
        printf("pid1 = %d", pid1); // B
    }
    else{
        pid1 = getpid();
        printf("pid = %d", pid); // C
        printf("pid1 = %d", pid1); // D
        wait(NULL);
    }
    return 0;
}   // Listing 2
```
##### Réponse
Ligne  A  --> 0
Ligne  B  --> 2603
Ligne  C  --> 2603
Ligne  D  --> 2600
#### Exercice 03
On considère le programme Listing 3. Expliquer la sortie au niveau de la ligne A.
```c
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int value = 5;

int main() {
    pid_t pid, pid1;

    // fork a child process
    pid = fork();

    if (pid == 0){
        value += 15;
        return 0;
    }
    else if(pid > 0){
        wait(NULL);
        printf("value = %d", value); // A
        return 0
    }
}   // Listing 2
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
b)
![exercice 4b](images/cours2/exercice4.png)

#### Exercice 05
Que fait le programme suivant:
```c
int main()
{
    int p=1;
    while(p>0)
        p=fork();
    execlp("prog", "prog", NULL);
    return 0;
}
```
##### Réponse
Avec ce programme, le processus initial (celui qui exécute main), crée continuellement des processus fils jusqu'à ce que
le système refuse de créer plus aucun autre processus (épuisement de ressource). Chacun des processus créés charge l'
image binaire d'un programme `prog` et l'exécute.

#### Exercice 06
On veut faire une recherche d'un élément dans un tableau d'entiers. Écrire un code ou le processus principal crée 4 processus fils qui chacun fait la recherche dans une partie du tableau. Chacun de ses processus fils exécute une fonction trouve (int element, int partie). Cette fonction retourne en cas de succès et 0 sinon. Un processus fils retourne le résultat de la recherche en utilisant l'appel système exit comme suit: `exit(0)` en cas de succès et `exit(1)` en cas d'échec. Quand le processus père est notifié du succès d'un processus fils, il tue tous les autres processus en faisant l'appel système `kill(pid, SIGKILL);`.
##### Réponse
```c
#include <sys/types.h>
#include <sys/wait.h>

int tab[12] = {12,7,0,4,7,21,3,100,103,99,54,17};

int trouve(int a, int partie){
  int j;

  for(j=partie*3; j < (partie+1)*3; j++)
    if(tab[j]==a){
      return 1;
    }

  return 0;
}

int main(){

	int pid[4], status, x, i;

	for(i = 0; i<4; i++)
	{
	  if((pid[i]=fork()) == 0){
		printf("Processus fils no %d cree : %d\n",i,getpid());
		if(trouve(4,i))
		  exit(0);
		else exit(1);
	  }
	}

	while((x=wait(&status))>0)
	   if(!WEXITSTATUS(status))
	   {
		 printf("Processus %d found it\n",x);
		 for(i=0;i<4;i++)
			if(pid[i]!=x) kill(pid[i],SIGKILL);
		  exit(0);
	   }
	exit(1);
}
```
#### Exercice 07
Écrivez un code ou le processus principal lance, l'un à la suite de l'autre, les fichiers exécutables passés en paramètres. Un fichier exécutable est lancé seulement quand tous ceux qui le précèdent sont terminés. Avant de se terminer, le processus principal affiche le nombre de lancement de fichiers exécutables qui ont échoués (parce que le fichier n'existe pas ou n'est pas exécutable).
##### Réponse
```c
#include<string.h>
#include<sys/types.h>
#include<sys/wait.h>
#include<stdlib.h>
#include<unistd.h>

int main(int argc, char* argv[])
{
  char* fich[argc-1];
  int status, i, j = 0;
  for(i=1;i<=argc;i++){
    if(fork()==0){
     execlp(argv[i],argv[i],NULL);
     exit(3);
    }

    wait(&status);
    if(WEXITSTATUS(status)==3)
    {
      fich[j]= (char*)malloc(strlen(argv[i])+1);
      strcpy(fich[j],argv[i]);
      j++;
    }
  }
  printf("Nombre de programmes noexecutes: %d\n",j);
  if(j!=0){
     printf("Liste des programmes non executes:\n");
     for(i=0;i<j;i++)
        printf("%s\n",fich[i]);
   }
  return 0;
}
```
#### Exercice 08
Écrire le code ou vous utiliser PThreads pour exprimer la concurrence. Ce programme permet de chercher le minimum dans un tableau d'entiers.
##### Réponse
```c
#include <pthread.h>
#include <limits.h>
#define MAX_THREADS	512
#define N_THREADS	10

void *find_min(void *list_ptr);


int min, partition_size = 2;
int list[N_THREADS*5] = {5, 2, 10, 6, 3, 9, 4, 23, 5, 6, 7, 2, 10, 17, 6, 4, 14, 72, 8, 9};

 main() {

  min = INT_MAX;
pthread_t p_threads[MAX_THREADS];
pthread_attr_t  attr;
int i;

  pthread_attr_init(&attr);
  pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);

for (i=0; i< N_THREADS; i++)
   pthread_create(&p_threads[i], &attr, find_min, (void *) &list[i*partition_size]);


for (i=0; i< N_THREADS; i++)
    pthread_join(p_threads[i], NULL);

printf("Minimum : %d\n", min);
 }

void *find_min(void *list_ptr) {

  int *local_ptr, i;

    local_ptr = (int *) list_ptr;

  for (i = 0; i < partition_size; i++)
        if (local_ptr[i] < min){

            min = local_ptr[i];
         }

 pthread_exit(0);
}
```

## Cours 3

## Cours 4
