# Writeups for NPST 2020 CTF

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
In our email today there was a file titled log.csv, seemingly containing password change requests for all workers. The email told us that there were reports of people being unable to log in, and we had to seartch through the ~5100 line file to see if we were able to find anything out of the ordinary. I did some experimenting by parsing the data in different ways using Python, and I kept doing this until I noticed something interesting. What gave me results was creating a Python script that printed out all the unique names, and how many times they occured in the log.

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