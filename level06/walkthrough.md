# **Level 06**

Comme d'habitude, decompilons l'executable pour y voir plus clair,
```
undefined4 auth(char *param_1,uint param_2)

{
  size_t sVar1;
  undefined4 uVar2;
  long lVar3;
  int local_18;
  uint local_14;
  
  sVar1 = strcspn(param_1,"\n");
  param_1[sVar1] = '\0';
  sVar1 = strnlen(param_1,0x20);
  if ((int)sVar1 < 6) {
    uVar2 = 1;
  }
  else {
    lVar3 = ptrace(PTRACE_TRACEME);
    if (lVar3 == -1) {
      puts("\x1b[32m.---------------------------.");
      puts("\x1b[31m| !! TAMPERING DETECTED !!  |");
      puts("\x1b[32m\'---------------------------\'");
      uVar2 = 1;
    }
    else {
      local_14 = ((int)param_1[3] ^ 0x1337U) + 0x5eeded;
      for (local_18 = 0; local_18 < (int)sVar1; local_18 = local_18 + 1) {
        if (param_1[local_18] < ' ') {
          return 1;
        }
        local_14 = local_14 + ((int)param_1[local_18] ^ local_14) % 0x539;
      }
      if (param_2 == local_14) {
        uVar2 = 0;
      }
      else {
        uVar2 = 1;
      }
    }
  }
  return uVar2;
}



// WARNING: Removing unreachable block (ram,0x0804889a)
// WARNING: Restarted to delay deadcode elimination for space: stack

bool main(void)

{
  int iVar1;
  int in_GS_OFFSET;
  char local_34 [32];
  int local_14;
  
  local_14 = *(int *)(in_GS_OFFSET + 0x14);
  puts("***********************************");
  puts("*\t\tlevel06\t\t  *");
  puts("***********************************");
  printf("-> Enter Login: ");
  fgets(local_34,0x20,stdin);
  puts("***********************************");
  puts("***** NEW ACCOUNT DETECTED ********");
  puts("***********************************");
  printf("-> Enter Serial: ");
  __isoc99_scanf();
  iVar1 = auth();
  if (iVar1 == 0) {
    puts("Authenticated!");
    system("/bin/sh");
  }
  if (local_14 != *(int *)(in_GS_OFFSET + 0x14)) {
                    // WARNING: Subroutine does not return
    __stack_chk_fail();
  }
  return iVar1 != 0;
}
```
On voit rapidement qu'il y a differentes `protections` nous empechons d'overflow ou de ret2libc par exemple.
On peut voir qu'il y a une execution de /bin/sh si le retour de auth est egal a 0.
Essayon de comprendre ce que fai la `fonction auth`,
Apres avoir pris nos 2 input, elle fait des calculs dessus, en retournant la `difference` entre,
le `second input` que l'on rentre et un `int calculer a partir de la string du premier input`,
Essayon de tester la fonction a part.
```
int auth(char *s, int a2)
{
  int i; // [esp+14h] [ebp-14h]
  int v4; // [esp+18h] [ebp-10h]
  int v5; // [esp+1Ch] [ebp-Ch]

  s[strcspn(s, "\n")] = 0;
  v5 = strnlen(s, 32);
  if ( v5 <= 5 )
    return 1;
  else
  {
    v4 = (s[3] ^ 0x1337) + 6221293;
    for ( i = 0; i < v5; ++i )
    {
      if ( s[i] <= 31 )
        return 1;
      v4 += (v4 ^ (unsigned int)s[i]) % 0x539;
      printf("v4 = %d\n", v4);
    }
    return a2 != v4;
  }
}

int main(int ac, char **av)
{
    printf("ret = %d\n", auth(av[1], atoi(av[2])));
}
```
```
./a.out test12 5      
v4 = 6227396
v4 = 6228348
v4 = 6228841
v4 = 6229860
v4 = 6230622
v4 = 6230838
ret = 1
```
Donc si input1 vaut `test12`, a la fin de boucle v4 vaut `6230838`.
Essayons donc avec ce chiffre en second parametre,
```
./a.out test12 6230838
v4 = 6227396
v4 = 6228348
v4 = 6228841
v4 = 6229860
v4 = 6230622
v4 = 6230838
ret = 0
```
Niquel, on a 0 retournons sur le level maintenant avec ces input.
```
level06@OverRide:~$ ./level06 
***********************************
*		level06		  *
***********************************
-> Enter Login: test12
***********************************
***** NEW ACCOUNT DETECTED ********
***********************************
-> Enter Serial: 6230838
Authenticated!
$ id
uid=1006(level06) gid=1006(level06) euid=1007(level07) egid=100(users) groups=1007(level07),100(users),1006(level06)
$ cat /home/users/level07/.pass
GbcPDRgsFK77LNnnuh7QyFYA2942Gp8yKj9KrWD8
```
> ### NEXT : [Level 07](/level07/walkthrough.md)
