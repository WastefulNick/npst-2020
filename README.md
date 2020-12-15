# Writeups for NPST 2020 CTF

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
Running this code will output `<=>?@ABCDEFGHIJKLMNOPQRSPST{BabyPenGwynDuhDuhDuhDuhDuhDuh}TSRQPONMLKJIHGFEDCBA@?>=`, if you remove the garbage, you get the flag `PST{BabyPenGwynDuhDuhDuhDuhDuhDuh}`.


## December 3rd
Around 19:00 on December 2nd everyone received a mail from our colleague, saying they guessed the password to the previously mentioned private.7z. The message was: `Jeg gjettet passordet til zip-fila,, og det funket!` (I guessed the passord to the zip-file,, and it worked!). You may think that the 2 commas are a spelling mistake, but the day afterwards, the 3rd, we received a new mail clarifying that the password to the zip-file was literally `til zip-fila,` (to the zip-file,).
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
def calculate_pask(aar):
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
This code would however only return the date of paaskeaften (easter evening), I still had to convert the date to an integer to get the maaltall. Having no clue about how the conversion worked, and not bothering taking the time to do so, I did the creative solution of using [W3 School's "Try it yourself" editor](https://www.w3schools.com/SQL/trysqlserver.asp?filename=trysql_func_sqlserver_convert) to run the SQl. Example: `SELECT CONVERT(int, CONVERT(DATETIME, '2040-03-31'));`.

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