# **Level 00**

Pour ce `level00` nous avons un executable lancer avec les droits du `user suivant` et notre but est que lors de l'execution du programme, nous puissions acceder au fichier .pass present dans le home du user suivant

Pour ce faire nous analysons le code de l'executable

> Voir [source](/level00/source)
```
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
	int local_14;
  
	puts("***********************************");
	puts("* \t     -Level00 -\t\t  *");
	puts("***********************************");
	printf("Password:");
	scanf("%d",local_14);
	if (local_14 != 5276) {
		puts("\nInvalid Password!");
	}
	else {
		puts("\nAuthenticated!");
		system("/bin/sh");
	}
	return local_14 != 5276;
}
```

Ce code est plutot simple, il regarde si l'entree utilisateur est `5276` et si c'est le cas, il lance un shell

```
level00@OverRide:~$ ./level00 
***********************************
* 	     -Level00 -		  *
***********************************
Password:5276

Authenticated!
$ cat /home/users/level01/.pass
uSq2ehEGT6c9S24zbshexZQBXUGrncxn5sD5QfGL
```

> ### NEXT : [Level 01](/level01/resources/README.md)
