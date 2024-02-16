# **Level 03**

Code decompiler:

```
int __cdecl decrypt(char a1)
{
  unsigned int i; // [esp+20h] [ebp-28h]
  unsigned int v3; // [esp+24h] [ebp-24h]
  char v4[29]; // [esp+2Bh] [ebp-1Dh] BYREF

  *(_DWORD *)&v4[17] = __readgsdword(0x14u);
  strcpy(v4, "Q}|u`sfg~sf{}|a3");
  v3 = strlen(v4);
  for ( i = 0; i < v3; ++i )
    v4[i] ^= a1;
  if ( !strcmp(v4, "Congratulations!") )
    return system("/bin/sh");
  else
    return puts("\nInvalid Password");
}
// 8048746: positive sp value 4 has been found

//----- (08048747) --------------------------------------------------------
int __cdecl test(int a1, int a2)
{
  int result; // eax
  char v3; // al

  switch ( a2 - a1 )
  {
    case 1:
    case 2:
    case 3:
    case 4:
    case 5:
    case 6:
    case 7:
    case 8:
    case 9:
    case 16:
    case 17:
    case 18:
    case 19:
    case 20:
    case 21:
      result = decrypt(a2 - a1);
      break;
    default:
      v3 = rand();
      result = decrypt(v3);
      break;
  }
  return result;
}

//----- (0804885A) --------------------------------------------------------
int __cdecl main(int argc, const char **argv, const char **envp)
{
  unsigned int v3; // eax
  int savedregs; // [esp+20h] [ebp+0h] BYREF

  v3 = time(0);
  srand(v3);
  puts("***********************************");
  puts("*\t\tlevel03\t\t**");
  puts("***********************************");
  printf("Password:");
  __isoc99_scanf("%d", &savedregs);
  test(savedregs, 322424845);
  return 0;
}
```
En comprenant le code, on voit que ce dernier nous invite a rentrer un "password",
qui sera utiliser dasn les fonction test et decrypt.
On peut voir aussi que la `fonction test` fait une `difference` entre les `2 parametres` qui sont,
`notre entree` et un `chiffre place en dur` dans le code, `322424845`.
De plus on voit aussi un switch case qui va donc de 1 a 21 par rapport a la difference.
Dans la fonction decrypt, on a une string, "Q}|u`sfg~sf{}|a3" qui encoder par rapport a notre inut.
Faisons donc un petit script pour decrypter tout ca,
```
n = 322424845
s = 'Q}|u`sfg~sf{}|a3'

for i in range(21):
    if chr(ord(s[0]) ^ i) == 'C':
        print(f"{i} : {chr(ord(s[0]) ^ i)}")
        break
print(f"Your input is {n - 18}")
```
```
python decrypt.py 
18 : C
Your input is 322424827
```
Allons tester cette input en password,
```
level03@OverRide:~$ ./level03 
***********************************
*		level03		**
***********************************
Password:322424827
$ id
uid=1003(level03) gid=1003(level03) euid=1004(level04) egid=100(users) groups=1004(level04),100(users),1003(level03)
$ cat /home/users/level04/.pass
kgv3tkEb9h2mLkRsPkXRfc2mHbjMxQzvb2FrgKkf
```
> ### NEXT : [Level 04](/level04/resources/README.md)
