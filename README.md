# Writeup for NPST 2020 CTF

## Preface
I just want to start of by saying that this is my first ever writeup, and the second CTF I have ever participated in (the first being NPST 2019). It being my first writeup means that I don't really have any clue what I'm doing, and I'm just writing whatever comes to mind. So please bear this in mind if you do decide to read through. 
I will also be presenting how I solved the challenges, and not the fine-polished, perfect ways to do it. This is because in a CTF where your ranking is based on completion time, a messy, yet quick solution, is what gets you to the top. 

## December 1st
On the first day we receive an email on the website with a verification code `RUV{JgkJqPåGtFgvLwnKilgp}`. Our supervisor had however dropped this code in their salad, so we had to decrypt it. The code can easily be recognized to be caesaer cipher, as hinted by it being dropped in the salad. Upon decrypting it with a shift of 2, you received the first flag: `PST{HeiHoNåErDetJulIgjen}`

## December 2nd
On the second day we received a zip folder confiscated from the secret agent Pen Gwyn. Inside the folder was a MID file, along with a 7z archive named `private`. Solving this included some trial and error, but in the end what had to be done was getting the integer values of the notes in the file, and converting these integers to ASCII. 
There were multiple ways to do this, some people simply opened the file in Notepad, manually removed garbage data, and saw the flag. I personally solved this by using the Python library mido:

```
from mido import MidiFile

out = ''
for x in MidiFile('pen_gwyn_greatest_hits.mid'):
    if x.type == 'note_on':
        out += chr(x.note)
        
print(out)
```
Running this code will output `<=>?@ABCDEFGHIJKLMNOPQRSPST{BabyPenGwynDuhDuhDuhDuhDuhDuh}TSRQPONMLKJIHGFEDCBA@?>=`. If you remove the garbage data, you get the flag `PST{BabyPenGwynDuhDuhDuhDuhDuhDuh}`.


## December 3rd
Around 19:00 on December 2nd everyone received a mail from our colleague, saying they guessed the password to the previously mentioned private.7z. The message was: `Jeg gjettet passordet til zip-fila,, og det funket!` (`I guessed the passord to the zip-file,, and it worked!`). You may think that the 2 commas are a spelling mistake, but the day afterwards, the 3rd, we received a new mail clarifying that the password to the zip-file was literally `til zip-fila,` (`to the zip-file,`).
After extracting the zip file, there were 2 files inside. The file `cupcake.png`, and a mysterious `kladd.txt`.

![Cupcake](cupcake.png)

In our daily mail we were told to analyze the image, to see if we could find anything out of the ordinary. I used the tool zsteg, and was able to find this
```
$ zsteg 3/cupcake.png
b1,rgb,lsb,xy       .. text: "youtu.be/I_8ZH1Ggjk0"
```
On the NPST website there was a file uploader called forbedre (enhance) that allowed you to upload images. [The YouTube link](youtu.be/I_8ZH1Ggjk0) lead to a video of the famous CSI enhance scene, which was a hint to use the enhance feature on the website on the image.

![Enhance!](enhance.gif)

After a bit of squinting, I was able to make out the flag `PST{HuskMeteren}`.

## December 4th
On this day we were told to find some info about Easter, and were told to find the sum of all values in the column Maaltall between 2020 and 2040. Included was a zip-file, with 4 SQL files and a .csv file. The .csv file, seemingly generated from the sql files, had a column for maaltall, but it was missing a few years in the period 2020-2040. I have close to no prior experience with SQL, but it looked like they generated maaltall reports for a given year. The file `dbo.FunctionPaaskeAften.sql` was the most interesting one, as it returned the variable `@paaskeaften`, which was then converted to an INT: `SELECT @maaltall = CONVERT(INT, @paaskeaften);`

Snippet from `dbo.FunctionPaaskeAften.sql`
```
--- Calculation steps:
SELECT @a=@aar%19, @b=FLOOR(1.0*@aar/100), @c=@aar%100;
SELECT @d=FLOOR(1.0*@b/4), @e=@b%4, @f=FLOOR((8.0+@b)/25);
SELECT @g=FLOOR((1.0+@b-@f)/3);
SELECT @h=(19*@a+@b-@d-@g+15)%30, @i=FLOOR(1.0*@c/4), @k=@aar%4;
SELECT @l=(32.0+2*@e+2*@i-@h-@k)%7;
SELECT @m=FLOOR((1.0*@a+11*@h+22*@l)/451);
SELECT @year = DATEADD(yy, @aar-2000, '2000/01/01');
SELECT @month = DATEADD(mm, FLOOR((1.0*@h+@l-7*@m+114)/31)-1, @year);
SELECT @easterday = DATEADD(dd, (@h+@l-7*@m+114)%31, @month);
SELECT @paaskeaften = CONVERT(DATETIME, @easterday) - 1;
```

Not having enough experience with SQL to be able to run this, I simply rewrote it in Python. Even though it isn't the prettiest solution, it was quick, simple, and gave results.
```
def calculate_pask(aar): # Takes year as input
    a = aar % 19
    b = floor(1.0*aar/100)
    c = aar % 100
    d = floor(1.0*b/4)
    e = b%4
    f=floor((8.0+b)/25)
    g=floor((1.0+b-f)/3)
    h=(19*a+b-d-g+15)%30
    i=floor(1.0*c/4)
    k=aar%4
    l=(32.0+2*e+2*i-h-k)%7
    m=floor((1.0*a+11*h+22*l)/451)
    month=floor((1.0*h+l-7*m+114)/31)-1
    easterday = (h+l-7*m+114)%31

    print(f'{year}-0{month+1}-{floor(easterday)}')

for x in range(2020, 2041):
    calculate_pask(x)
```
This code would however only return the date of paaskeaften (easter evening), I still had to convert the date to an integer to get the maaltall. Having no clue about how the conversion worked, and not having the time to learn, I did the creative solution of using [W3 School's "Try it yourself" editor](https://www.w3schools.com/SQL/trysqlserver.asp?filename=trysql_func_sqlserver_convert) to run the SQl. Example: `SELECT CONVERT(int, CONVERT(DATETIME, '2040-03-31'));`.

I then manually fixed the .csv file, and made a short script to print the sum of all the maaltall values.
```
with open('DatoPaaskeMODIFIED.csv', 'r') as f:
    lines = f.readlines()

sum = 0
for x in lines:
    sum += int(x.split(';')[-1])

print('PST{'+str(sum)+'}')
```
Running this code with the proper Maaltall values for all years would print the flag `PST{999159}`.

## December 5th
In our email today there was a file titled log.csv, seemingly containing password change requests for all workers. The email told us that there were reports of people being unable to log in, and we had to search through the ~5100 line file to see if we were able to find anything out of the ordinary. I did some experimenting by parsing the data in different ways using Python, and I kept doing this until I noticed something interesting. What gave me results was creating a Python script that printed out all the unique names, and how many times they occured in the log.

```
from urllib.parse import unquote

log = open('log.csv', 'r', encoding='utf-16')

unique_names = {}

for line in log.readlines():
    name = unquote(line).split(';')[1].split('+')[0]
    
    if name in unique_names:
            unique_names[name] += 1
    else:
        unique_names[name] = 1

print(dict(sorted(unique_names.items(), key=lambda item: item[1])))
```
Output: `{'Ni\u200bssen': 1, 'Nissen': 4, 'Alf': 17, ... `

After running it I noticed a name with only one occurence, which also contained a zero width space in it (\u200b).

I then simply opened the log file, used VS Code's regex ctrl + f to search for `N.*ssen`. There were 5 results, the 5th being `Ni%E2%80%8Bssen+%3CJule+Nissen%3E`.
If you URL decoded the whole line, it looked like this: `2020-10-1508:35:03;Ni​ssen+<Jule+Nissen>;SPF+<Seksjon+for+Passord+og+Forebygging>;I+dag+har+jeg+lyst+til+at+PST{879502f267ce7b9913c1d1cf0acaf045}+skal+være+passordet+mitt`

The flag was the password this user had requested to change to, which was `PST{879502f267ce7b9913c1d1cf0acaf045}`

## December 6th
On this day we would be starting with e-learning in [SLEDE8](https://slede8.npst.no/), which is an assembly language created for the CTF. [Here](https://github.com/PSTNorge/slede8/blob/main/README.md) is the documentation for the assembly language. We were told to open the e-learning module using the code `4032996b1bbb67f6`, and upon doing so a task popped up.

```
Første byte med føde er et tall N som representerer antallet påfølgende bytes med føde.
(The first byte of feed (input) is a number N which represents the number of following bytes of feed)
Beregn summen av de N påfølgende tallene, og gi resultatet som oppgulp.
(Calculate the sum of the N following numbers, and give the result as regurgitation (output))

Lykke til! (Good luck!)
```

Here is the code I ended up writing:
```
LES r2         ; Reads the first number of the feed, this is amount of following bytes.
SETT r15, 0x01 ; This is how much the loop will increment with every time (1)

loop:
LIK r2, r3     ; If r2 (N feed) = r3 (# of feed read) then it will
BHOPP Stopp    ; jump to the end of the code.
PLUSS r3, r15  ; This will increment the counter, r3, by the amount stored in r15 (0x01)

LES r0         ; Reads the feed into r0
PLUSS r1, r0   ; r1 (final int) += r0 (feed input)
HOPP loop      ; Jumps to the top of the loop

Stopp:
SKRIV r1       ; Writes the results to the regurgitation
STOPP          ; Stops
```
If for example given the feed `0401020304`, the program would then return `0a` (1+2+3+4=10).
I sent in the assembly, and the server responded with a success and the flag: `PST{ATastyByteOfSled}`

## December 7th
On this day we were told that a weird signal was being picked up at santa's workshop, and it was our task to find out what it was. Included was the file `data.complex16u`. This challenge opened up for a pretty huge rabbit hole, as running the Unix [`file` command](https://en.wikipedia.org/wiki/File_%28command%29) on the program would return `data.complex16u: International EBCDIC text, with very long lines, with no line terminators`. This sent a lot of people down a rabbit hole, including myself, but to no luck. 
After spending a little while trying to decode the apparent EBCDIC file, I started searching for `complex16u` instead. One of the search results was [this GitHub pull request](https://github.com/jopohl/urh/pull/772), which added support for complex16u files to the program Universal Radio Hacker. I downloaded the program and opened the data file in it. It converted the signal to bits, and if you did a binary to ASCII conversion you got `ªªªª PST{0n_0ff_k3y1ng_1s_34sy!} `. The flag was `PST{0n_0ff_k3y1ng_1s_34sy!}`

## December 8th
In this email we were told that achademic development is important in these pre-Christmas times, and today's theme was [`ASN.1`](https://en.wikipedia.org/wiki/ASN.1).
Included was this string `MIIBOTCCATAwggEnMIIBHjCCARUwggEMMIIBAzCB+zCB8zCB6zCB4zCB2zCB0zCByzCBwzCBuzCBszCBqzCBozCBnDCBlDCBjDCBhDB9MHYwbzBoMGEwWjBTMEwwRTA+MDcwMTAqMCMwHDAVMA4wBwUAoQMCAROgAwIBA6EDAgEMogMCAQChAwIBE6ADAgEBoQMCARKkAgUAoQMCARShAwIBDqIDAgEYoQMCAQShAwIBEqEDAgEOoQMCAQ6hAwIBB6IDAgECogMCAQigAwIBAaIDAgENogMCARKiAwIBAKMCBQCiAwIBE6IDAgESogMCAQ+hAwIBEaEDAgEOoQMCAQugAwIBAKIDAgEDoQMCAQyhAwIBFKEDAgESoQMCAQ+gAwIBAaEDAgEMoAMCAQOhAwIBEaEDAgEOogMCAQs=` and this schema:
```
Spec DEFINITIONS ::= BEGIN
    LinkedList ::= Node
    Node ::= SEQUENCE {
        child CHOICE {
            node Node,
            end NULL
        },
        value CHOICE {
            digit                [0] INTEGER(0..9),
            lowercase           [1] INTEGER(0..25),
            uppercase           [2] INTEGER(0..25),
            leftCurlyBracket    [3] NULL,
            rightCurlyBracket   [4] NULL
        }
    }
END
```
Pretty much none of the solutions I saw to this challenge were pretty, mine definitely wasn't. I tried using Python's [asn1tools](https://asn1tools.readthedocs.io/en/latest/) to solve it, but I had some problems with decoding the schema. In any other situation the best thing to do would be to read the documentation, but being in a CTF and knowing there was a quicker way, I did a tiny workaround. I noticed that the tools included [a shell](https://asn1tools.readthedocs.io/en/latest/#the-shell-subcommand), and I used this to get the data I needed.

```
$ compile schema.asn
$ convert Node 3082013930820130308201273082011e308201153082010c308201033081fb3081f33081eb3081e33081db3081d33081cb3081c33081bb3081b33081ab3081a330819c30819430818c308184307d3076306f30683061305a3053304c3045303e30373031302a3023301c3015300e30070500a103020113a003020103a10302010ca203020100a103020113a003020101a103020112a4020500a103020114a10302010ea203020118a103020104a103020112a10302010ea10302010ea103020107a203020102a203020108a003020101a20302010da203020112a203020100a3020500a203020113a203020112a20302010fa103020111a10302010ea10302010ba003020100a203020103a10302010ca103020114a103020112a10302010fa003020101a10302010ca003020103a103020111a10302010ea20302010b
```
The file schema.asn was simply the schema we received in the mail. The second line is the input, but [base64decoded, then converted to hex using cyberchef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)To_Hex('None',0)&input=TUlJQk9UQ0NBVEF3Z2dFbk1JSUJIakNDQVJVd2dnRU1NSUlCQXpDQit6Q0I4ekNCNnpDQjR6Q0IyekNCMHpDQnl6Q0J3ekNCdXpDQnN6Q0JxekNCb3pDQm5EQ0JsRENCakRDQmhEQjlNSFl3YnpCb01HRXdXakJUTUV3d1JUQStNRGN3TVRBcU1DTXdIREFWTUE0d0J3VUFvUU1DQVJPZ0F3SUJBNkVEQWdFTW9nTUNBUUNoQXdJQkU2QURBZ0VCb1FNQ0FSS2tBZ1VBb1FNQ0FSU2hBd0lCRHFJREFnRVlvUU1DQVFTaEF3SUJFcUVEQWdFT29RTUNBUTZoQXdJQkI2SURBZ0VDb2dNQ0FRaWdBd0lCQWFJREFnRU5vZ01DQVJLaUF3SUJBS01DQlFDaUF3SUJFNklEQWdFU29nTUNBUStoQXdJQkVhRURBZ0VPb1FNQ0FRdWdBd0lCQUtJREFnRURvUU1DQVF5aEF3SUJGS0VEQWdFU29RTUNBUStnQXdJQkFhRURBZ0VNb0FNQ0FRT2hBd0lCRWFFREFnRU9vZ01DQVFzPQ).
Running the last command in the shell would print out a long string, something along the lines of:
```
        },
        value digit : 3
    },
    value lowercase : 12
},
value uppercase : 0
```
I then copy-pasted this long string into a text file called `shell_out.txt`, and then created an ugly Python script using regex to parse it.
```
import string
import re

test = open('shell_out.txt', 'r').read()
final = ''

for a in re.findall(r'value ([a-zA-Z]*) : ([0-9]+|NULL)', test):
    if a[0] == 'lowercase':
        final += string.ascii_lowercase[int(a[1])]
    if a[0] == 'uppercase':
        final += string.ascii_uppercase[int(a[1])]
    if a[0] == 'digit':
        final += a[1]
    if 'left' in a[0]:
        final += '{'
    if 'right' in a[0]:
        final += '}'

print(final[::-1])
```
Output: `Lor3m1psumD0lorPST{ASN1IChooseYou}s1tAm3t`

The flag is `PST{ASN1IChooseYou}`

## December 9th
Today NPST had received a chat log from an SPST agent, which included a worrying amount of emojis. The message is obfuscated, and they're wondering if the message could possibly be HEXMAS encoded. Message:
```
🎅🤶❄⛄🎄🎁🕯🌟✨🔥🥣🎶🎆👼🦌🛷

🤶🛷✨🎶🎅✨🎅🎅🛷🤶🎄🔥🎆🦌🎁🛷🎅❄🛷🛷🎅🎶🎅✨🎅🦌🥣🔥🛷🦌⛄🎅🌟🛷🛷🔥🎄🦌🎅✨🦌🦌🕯🎶🎅🤶🦌❄🎁🕯🎅✨🎶👼🌟🎆🕯🌟❄👼🎅🎅🤶❄🎄👼🎆🔥🎁🛷🤶👼🎅🎅🎅🎅🎅🎅
```
The first thing I did was to do some simple analysis in Python. After printing out random stuff, I decided to print out how many unique emojis there are; 16. I should have understood this sooner, from the HEXMAS hint, but better late than never. On the first line were 16 emojis, so I guessed that these were the keys. 🎅=0x1, 🤶0x2, ..., 🛷=0xf.

I then made a short Python script to decode the message.
```
hexmas = open('input.txt', 'r', encoding='utf-8').read().split('\n')

lookup = hexmas[0]
message = hexmas[-1]

message_hex = ''.join([hex(list(lookup).index(x))[2:] for x in message])
print(message_hex)
```
The output of this was `1f8b0800f149ce5f02ff0b080ea9fe307ff94e08ee6b01e25608bd7c672d00124dc95f1d000000`. Translating this from hex did however not return any useful data. Luckily [CyberChef's](https://gchq.github.io/CyberChef/#recipe=Magic(3,false,false,'')&input=MWY4YjA4MDBmMTQ5Y2U1ZjAyZmYwYjA4MGVhOWZlMzA3ZmY5NGUwOGVlNmIwMWUyNTYwOGJkN2M2NzJkMDAxMjRkYzk1ZjFkMDAwMDAw) magic managed to find out that it was gunzip compressed. Converting the hex string from hex, and then gunzip decompressing it gives the flag: `PST{🧹🧹🎄🎅🎄🧹}` .

## December 10th
On this day we received yet another Slede8 task to solve.
```
Føde består av to tall, A og B (The feed contains 2 numbers, A and B)
Skriv ut resultatet av (A + B) mod 256 som en ASCII-streng (Write the result of A + B mod 256 as an ASCII-string)

Eksempel: A=0xA0 og B=0x08 => '168'
Eksempel: A=0xFF og B=0xFF => '254'
```
Because of the modulo 256 you could ignore overflows, which made it a bit easier. The challenge with this was converting hex -> decimal, to be able to print out the number in ASCII. A good way to do this, is working your way down. For example given the input `a0e6`, you would then find the sum of these (%256): `0x86` (134 in base10). You would then check that if the number is larger than 200, since it's not it will check if it's larger than 100. Since it is, add 1 to the `hundreds` register. Then subtract 100 from the value, 134-100=34. Then loop over and subtract 10's from it, until the value is less than 10. After looping 3 times, the remaining value will be 4. The `hundred`, `tens` and `ones` register will respectively hold; `0x01`, `0x03` and `0x04`. Since the ASCII value of `0` is `0x30`, we can just add `0x30` to all the registers, the program will then print out: 313334 (0x31, 0x33, 0x34), or in ASCII; 134.
```
LES r0
LES r1
PLUSS r0, r1

SETT r2, 0x64
SETT r3, 0xc8
SETT r4, 0x0a
SETT r6, 0x01

SEL r0, r3
BHOPP set200

SEL r0, r2
BHOPP set100

SETT r7, r0
PLUSS r7, r4
HOPP tens

set100:
SETT r10, 0x01
MINUS r0, r2
SETT r7, r0
PLUSS r7, r4
HOPP tens

set200:
SETT r10, 0x02
MINUS r0, r3
SETT r7, r0
PLUSS r7, r4
HOPP tens

tens:
PLUSS r5, r4
SEL r0, r5
PLUSS r11, r6
MINUS r7, r4
BHOPP tens
MINUS r11, r6

SETT r12, r7
SETT r8, 0x30

PLUSS r10, r8
PLUSS r11, r8
PLUSS r12, r8

SETT r15, 0x30

LIK r10, r15
BHOPP removezeroes

SKRIV r10

under100:
SKRIV r11

under10:
SKRIV r12

STOPP

removezeroes:
LIK r11, r15
BHOPP under10
HOPP under100
```
Upon submitting this code, we receive the flag `PST{++AndKissesWillBeAwardedToYou}`.

## December 11th
In our daily mail we were told that NPST had noticed an unauthorized change to Santa's naughty and nice list. An md5 sum had been modified, but they needed our help to find out which. Attached was a zip file, `liste.zip`. In the zip file was 3 SQLite database files. The only important file was the `liste.db` file. Using [`sqlitebrowser`](https://sqlitebrowser.org/) to open the file, I saw the 2 tables `snille` (nice) and `slemme` (naughty). The files were in the format `firstname:lastname:md5hash`. I did some quick tests, and found out that the md5hash was calculated by `md5(firstnamelastname)`. Since there was a modified md5 hash, it would be easy to enumerate every line in the file, and check if the md5hash actually matched the md5 of the first, lastname. I exported the files as .csv files, and I wrote a short Python script to check if all md5 hashes were correct.
```
from hashlib import md5

with open('snille.csv', 'r', encoding='utf-8') as f:
    for line in f.readlines()[1:]:
        rows = line.strip().split(';')
        hashed = md5(f'{rows[0]}{rows[1]}'.encode()).hexdigest()
        if hashed != rows[-1]:
            print(f'PST{{{rows[2]}}}')
```
In the nice list, there was 1 md5 that didn't match the md5 of first, last. This hash, wrapped in `PST{}` was the flag: `PST{49422712408d5409a3e40945204314e6}` 

## December 12th
SPST had allegedly posted some s8asm code on their GitHub, which was luckily saved by our colleague before it was deleted. Attached was the file `program.s8`, which was assembled Slede8 code. The first step was to disassemble the file into working s8asm. I had already created an s8 disassembler previously (which I will not be sharing). Slede8 doesn't have a lot of opcodes, so it's very simple to create a disassembler. My disassembler gave me this code as output (manually changed function names):
```
SETT r5, 0x00                           ;0x0
SETT r6, 0x01                           ;0x2
SETT r10, 0x00                          ;0x4
SETT r11, 0x01                          ;0x6
SETT r12, 0x00                          ;0x8
FINN xor_data                           ;0xa
SETT r9, 0x1a                           ;0xc

decrypt_loop:
LES r2                                  ;0xe
LAST r3                                 ;0x10
SETT r7, r5                             ;0x12
PLUSS r7, r6                            ;0x14
XELLER r2, r7                           ;0x16
XELLER r2, r3                           ;0x18
ELLER r12, r2                           ;0x1a
SETT r5, r6                             ;0x1c
SETT r6, r7                             ;0x1e
MINUS r9, r11                           ;0x20
PLUSS r0, r11                           ;0x22
ULIK r9, r10                            ;0x24
BHOPP decrypt_loop                      ;0x26

LIK r12, r10                            ;0x28
BHOPP correct                           ;0x2a

FINN failed_string                      ;0x2c
TUR write_output                        ;0x2e
STOPP                                   ;0x30

correct:
FINN correct_string                     ;0x32
TUR write_output                        ;0x34
STOPP                                   ;0x36

xor_data:
.DATA 0x51,0x51,0x57,0x7e,0x6e,0x64,0x77,0x12,0x59,0x38,0xf3,0x8a,0x48,0x3d,0xeb,0x53,0x7d,0x21,0x5c,0xaf,0x1c,0xae,0x50,0x25,0x55,0x3f

correct_string:
.DATA 0x4b,0x6f,0x72,0x72,0x65,0x6b,0x74,0x21

failed_string:
.DATA 0x00,0x46,0x65,0x69,0x6c,0x21,0x00

write_output:
LAST r2                                 ;0x62
LIK r2, r10                             ;0x64
BHOPP return                            ;0x66
SKRIV r2                                ;0x68
PLUSS r0, r11                           ;0x6a
HOPP write_output                       ;0x6c

return:
RETUR                                   ;0x6e
```
By looking at the code, our goal was to jump to the address 0x32, which would then print `Korrekt!` (Correct!). On line 0x28 it checked if the register `r12`'s value was 0x00. So to get print out the `correct_string` we had to keep r12 to 0x00. r12 was only modified on line 0x1a, where it was ORed with r2. For r12 to be 0x00, r2 had to also be 0x00. For r2 to be 0x00, we have to look at the line above, where r2 is XORed with r3. One of the properties of XOR, is that if you XOR a value with itself, it will always become 0. Our current goal was then to get r2 to be the same as r3. r2's value was determined by the feed input XORed with r7. We then had the equation: x ^ r7 = r3. Where x is the feed (r2). I could have implemented the entire `decrypt_loop` in Python, but I was too tired to do this. Instead I wrote a short Python script to find `x` if you knew r7 and r3.
```
r3 = ''
r7 = ''

result = int(r3, 16)
to_xor = int(r7, 16)

for x in range(0, 255):
    if x ^ to_xor == result:
        print(hex(x))
        break
```
I then proceeded to painstakingly find r3 and r7 for all of the 26 feed bytes by looking at the register and manually filling it out in the Python script. After torturing myself for what felt like an eternity, I finally got the complete feed: `5053547b666962306e616363315f306e6574316d335f7034647d` which when ran with the program printed `Korrekt!`. I then converted the hex to ASCII, and got the flag: `PST{fib0nacc1_0net1m3_p4d}`.

## December 13th
NPST had received a message over fax, but no one in the office understood the message. It appeared to be hex-encoded, but hex-decoding it didnt give anything meaningful. Included was the file `melding.txt`. At first glance the challenge reminded me of one they had in last year's CTF, titled `Linebreak it till you make it`. It was a file with 0's and 1's, and when resized the correct way and ctrl + f'ing for 1, you would see the flag. This did however not work for this challenge, I tried ctrl + f for all hex values, with no result. I then spent probably hours reading up on fax, and trying to somehow convert the data to a fax document or similar. After a long break, I came back to the challenge. I looked at the output from a Python script I made at the start:
```
unique = {}

f = open('melding.txt', 'r').read()

for char in f:
    if char == '\n':
        pass
    elif char in unique:
        unique[char] += 1
    else:
        unique[char] = 1

print(sorted(unique.items(), key=lambda item: item[1]))
```
The output looked like this: `[('D', 35), ('8', 39), ('0', 41), ('B', 42), ('9', 46), ('A', 47), ('6', 49), ('4', 56), ('3', 160), ('2', 178), ('C', 180), ('E', 183), ('5', 189), ('1', 189), ('F', 192), ('7', 194)]`. I had noticed that half of the characters appeared around 35-55 times, while the other half appeared around 160-195 times. Thinking back to the last year's linebreak task, I decided to try something incredibly stupid. Using Visual Studio Code's regex search, I searched for the 8 least occuring values in the message. Highlighted was the flag: `PST{SNEAKY_FLAG_IS_SNEAKY}`.

![Sneaky](sneaky_flag.png)

## December 14th
TODO: this

## December 15th
This challenge was very similar to the one on December 7th. Once again we received a complex16u file, this time titled `data2.complex16u`. After opening it in URH I spent some time clicking around, but it seemed that no matter what I did, the binary representation of the signal was nonsense. After a while of clicking I found the `Decoding` tab (under Edit). I pasted in the binary string URH gave me for the signal, and tried all of the different decoding methods. `Manchester II` decoding gave the flag: `PST{m4nch3st3r_3nc0d1ng_1s_4_l0t_0f_fun!}`.

## December 16th
TODO: this

## December 17th
This day was quite similar to the 8th, but of course a tiny bit harder. We were told that they had been listening to an SPST's agent's phone, and that the network operator had sent data according to ETSI232-1. [ETSI232-1 is a standard for Lawful Interception (LI); Handover Interface and Service-Specific Details (SSD) for IP delivery;](https://www.etsi.org/deliver/etsi_ts/102200_102299/10223201/03.20.01_60/ts_10223201v032001p.pdf). Included were 2 files, `ETSI232-1.txt` (an ASN.1 schema) and `data.b64.txt` (base64 encoded data). This time I actually knew how asn1tools worked, and I was able to make it properly decode the data.b64 according to the schema. I used this Pyhthon script to decode:
```
import asn1tools
from base64 import b64decode

schema = asn1tools.compile_files('ETSI232-1.txt')
b64data = open('data.b64.txt', 'r').read()

data = schema.decode('PS-PDU', b64decode(b64data))
```
Data would then be a dict of the decoded data. The interesting part of the dict was the payload part, where the messages were. It looks something like this:
```
{
    payloadDirection fromTarget,
    cCContents undefinedCC : '634B40044F5241484005'H
},
{
    payloadDirection fromTarget,
    cCContents undefinedCC : '6B5241560A'H
},
{
    payloadDirection toTarget,
    cCContents undefinedCC : '6C414D0A04'H
},
{
    payloadDirection toTarget,
    cCContents undefinedCC : '6C45560440510442514A4A4150044A4B410443E79C5D1B'H
},

...
```
I assumed that the the hex strings were the messages, but they didn't decode into ASCII. I did some experimenting in [CyberChef](https://gchq.github.io/CyberChef/), and using the XOR bruteforce, I noticed that XORing every character of the hex decoded string with 0x24 would give what I was looking for. [Example](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')XOR_Brute_Force(1,100,0,'Standard',false,true,false,'')&input=NkM0NTU2MDQ0MDUxMDQ0MjUxNEE0QTQxNTAwNDRBNEI0MTA0NDNFNzlDNUQxQg).

I then made a short Python script that did this for every message.
```
import asn1tools
from base64 import b64decode

schema = asn1tools.compile_files('ETSI232-1.txt')
b64data = open('data.b64.txt', 'r').read()

data = schema.decode('PS-PDU', b64decode(b64data))

for x in data['payload'][1]:
    direction = x['payloadDirection']
    author = direction.replace('fromTarget', 'Pen Gwyn:').replace('toTarget', 'SPST HQ?:')
    hex_data = x['cCContents'][1]

    print(author, ''.join([chr(y ^ 0x24) for y in hex_data]))
```

This would then output the (rather funny) conversation, but there was no flag in it. The interesting part of the conversation was the last part:
```
...

Pen Gwyn: d9c36ccf
SPST HQ?: hæ? - (What?)
Pen Gwyn: 6a38
Pen Gwyn: 4281
Pen Gwyn: b48f
SPST HQ?: ????
Pen Gwyn: d14db694daae
SPST HQ?: Hva ser jeg på - (What am I looking at?)
Pen Gwyn: Det skal være en uuid. - (It's supposed to be a uuid)
Pen Gwyn: Bindestrekknappen min funker ikke - (My dash-button isn't working)
SPST HQ?: Og hva godt skal det gjøre meg? - (And what good am I supposed to do with this?)
Pen Gwyn: Du må ta md5 av uuid'en som lowercase hex og legge til det vanlige. - (You have to take the md5 of the uuid as a lowercase hex, and add the usual)
SPST HQ?: Skjønner! - (I understand!)
SPST HQ?: Det funker ikke ... - (It doesn't work..)
Pen Gwyn: Whoops. Uuiden skulle starte med c9c(...) - (Whoops. The uuid is supposed to start with c9c(...))
Pen Gwyn: ... og slutte med (...)4a3 - (... and end with (...)4a3)
SPST HQ?: WIN! Takk. - (WIN! Thanks)
Pen Gwyn: Under og inn - (Under and in)
```
Since Pen Gwyn's dash-button was broken, he sent the uuid in seperate messages, which combined gives: `d9c36ccf-6a38-4281-b48f-d14db694daae`. As the last few messages said, there was an error, and you had to replace the start with c9c and the end with 4a3; `d9c36ccf-6a38-4281-b48f-d14db694daae`. I then got the md5 of this string
```
$ echo -n c9c36ccf-6a38-4281-b48f-d14db694d4a3 | md5sum
0ae06caf767ac7ebce290cfc57be6a6f  -
```
The flag was then `PST{0ae06caf767ac7ebce290cfc57be6a6f}`

## December 18th
TODO: this

## December 19th
In today's email we were explained that the elf Sigurd had created a way to split a secret into X keys. Further, the algorithm is made in such a way where only a Y amount of keys is required to get the secret. The secret password to the Christmas present vault was split into 5 (X) keys, of which 3 (Y) of them were required to open the vault. Two of the keys were given to Santa Claus, 1 key to elf Reidar, 1 key to elf Sigurd and 1 key to elf Adrian. This means that the vault can either be opened by Santa Claus + one elf, or by all 3 elves. Santa Claus has however lost both his keys, it is necessary to get back the vault's secret. We are also told that Sigurd's favorite number is `6864797660130609714981900799081393217269435300143305409394463459185543183397656052122559640661454554977296311391480858037121987999716643812574028291115057151` and we were given the 3 secrets held by the elves:

Reidar: (3, 570999082059702856147787459046280784390391309763131887566210928611371012340016305879778028495709778777)

Sigurd: (4, 922383132557981536854118203074761267092170577309674587606956115449137789164641724882718353723838873409)

Adrian: (5, 1361613195680829887737031633110361870469394661742852962657887598996346260195423498636393760259000241699)

Using a well-crafted search, I was quickly able to find out that the method used was probably [`Shamir's Secret Sharing`](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing). Luckily there was a [Python example on the Wikipedia page](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing#Python_example), so I copied this and modified it a bit, and made it spit out the flag.
```
import codecs

def _extended_gcd(a, b):
    x = 0
    last_x = 1
    y = 1
    last_y = 0
    while b != 0:
        quot = a // b
        a, b = b, a % b
        x, last_x = last_x - quot * x, x
        y, last_y = last_y - quot * y, y
    return last_x, last_y

def _divmod(num, den, p):
    inv, _ = _extended_gcd(den, p)
    return num * inv

def _lagrange_interpolate(x, x_s, y_s, p):
    k = len(x_s)
    assert k == len(set(x_s)), "points must be distinct"
    def PI(vals):  # upper-case PI -- product of inputs
        accum = 1
        for v in vals:
            accum *= v
        return accum
    nums = []  # avoid inexact division
    dens = []
    for i in range(k):
        others = list(x_s)
        cur = others.pop(i)
        nums.append(PI(x - o for o in others))
        dens.append(PI(cur - o for o in others))
    den = PI(dens)
    num = sum([_divmod(nums[i] * den * y_s[i] % p, dens[i], p)
               for i in range(k)])
    return (_divmod(num, den, p) + p) % p

def recover_secret(shares, prime):
    if len(shares) < 2:
        raise ValueError("need at least two shares")
    x_s, y_s = zip(*shares)
    return _lagrange_interpolate(0, x_s, y_s, prime)

prime = 6864797660130609714981900799081393217269435300143305409394463459185543183397656052122559640661454554977296311391480858037121987999716643812574028291115057151 
shares = [
    (3, 570999082059702856147787459046280784390391309763131887566210928611371012340016305879778028495709778777),
    (4, 922383132557981536854118203074761267092170577309674587606956115449137789164641724882718353723838873409),
    (5, 1361613195680829887737031633110361870469394661742852962657887598996346260195423498636393760259000241699)]

recovered = recover_secret(shares, prime)
print(codecs.decode(hex(recovered)[2:], 'hex').decode())
```
The flag is: `PST{f0rd3lt_4nsv4r_3r_d3t_b3st3_4nsv4r3t!}`