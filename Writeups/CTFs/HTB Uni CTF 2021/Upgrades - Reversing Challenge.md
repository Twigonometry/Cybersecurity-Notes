# Hack the Box Uni CTF 2021 - Upgrades

This weekend (19th - 21st November) HackTheBox hosted their second University CTF. This was the first I participated in, and it was full of interesting challenges. I would have loved to have spent more time on this CTF, but unfortunately (and somewhat ironically) due to heavy University workloads I didn't have much time (or energy) to dedicate.

Where I did put time into the CTF I tried my best to step out of my comfort zone and do some challenges in categories I'd never tackled before, like Crypto and Reverse Engineering. While I only completed one challenge in these categories, I learned an awful lot of cool new stuff that I'm sure will be useful when we build the SESH CTF this year.

Thanks to HTB for hosting this CTF and giving me the chance to learn some new skills post-OSCP. Here's my writeup of Upgrades, the first reversing challenge.

## Challenge Info

> We received this strange advertisement via pneumatic tube, and it claims to be able to do amazing things! But we there's suspect something strange in it, can you uncover the truth?

## Examining Files

I downloaded the `.zip` file and unzipped it:

```bash
┌──(kali㉿kali)-[~/Documents/htb/uni-ctf/reversing]
└─$ mkdir upgrades 
┌──(kali㉿kali)-[~/Documents/htb/uni-ctf/reversing]
└─$ mv ~/Downloads/rev_upgrades.zip upgrades 
┌──(kali㉿kali)-[~/Documents/htb/uni-ctf/reversing]
└─$ cd upgrades 
┌──(kali㉿kali)-[~/…/htb/uni-ctf/reversing/upgrades]
└─$ unzip rev_upgrades.zip 
Archive:  rev_upgrades.zip
   creating: rev_upgrades/
  inflating: rev_upgrades/Upgrades.pptm 
```

The first thing I checked was `strings` for readable ASCII characters. There was a lot, but nothing that looked obviously like a flag, so I searched for flag prefix:

```bash
┌──(kali㉿kali)-[~/…/uni-ctf/reversing/upgrades/rev_upgrades]
└─$ strings Upgrades.pptm | grep HTB
```

This found nothing.

I knew from previous Hack the Box challenges, such as [Patents](https://0xdf.gitlab.io/2020/05/16/htb-patents.html), that `.docx` files were actually just archives - I suspected the same applied to PPTM files. Kali's archive manager could not open the archive:

![[Pasted image 20211119172219.png]]

I could open it in Windows, however:

![[Pasted image 20211121203851.png]]

I received a nice warning about macros being enabled, and a clue about some automated processes. This made me sure I was looking in the right direction.

## Examining Macros

[This article](https://www.lifewire.com/pptm-file-2622189) confirmed the use of macros in `.pptm` files, and [this one](https://superuser.com/questions/661315/tools-to-extract-text-from-powerpoint-pptx-in-linux) suggests you can unzip them as normal (despite the Archive Manager struggling):

```bash
┌──(kali㉿kali)-[~/…/uni-ctf/reversing/upgrades/rev_upgrades]
└─$ unzip Upgrades.pptm    
Archive:  Upgrades.pptm
  inflating: [Content_Types].xml     
  inflating: _rels/.rels             
  inflating: ppt/presentation.xml    
  inflating: ppt/slides/slide1.xml   
  inflating: ppt/slides/slide2.xml   
  inflating: ppt/slides/slide3.xml   
  inflating: ppt/slides/slide4.xml   
  inflating: ppt/slides/_rels/slide4.xml.rels  
  inflating: ppt/slides/_rels/slide1.xml.rels  
  inflating: ppt/_rels/presentation.xml.rels  
  inflating: ppt/slides/_rels/slide3.xml.rels  
  inflating: ppt/slides/_rels/slide2.xml.rels  
  inflating: ppt/slideMasters/slideMaster1.xml  
  inflating: ppt/slideLayouts/_rels/slideLayout10.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout11.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout6.xml.rels  
  inflating: ppt/slideLayouts/slideLayout1.xml  
  inflating: ppt/slideLayouts/slideLayout2.xml  
  inflating: ppt/slideLayouts/slideLayout4.xml  
  inflating: ppt/slideLayouts/slideLayout5.xml  
  inflating: ppt/slideLayouts/slideLayout6.xml  
  inflating: ppt/slideLayouts/slideLayout7.xml  
  inflating: ppt/slideLayouts/slideLayout8.xml  
  inflating: ppt/slideLayouts/slideLayout9.xml  
  inflating: ppt/slideLayouts/slideLayout10.xml  
  inflating: ppt/slideLayouts/slideLayout11.xml  
  inflating: ppt/slideLayouts/slideLayout3.xml  
  inflating: ppt/slideMasters/_rels/slideMaster1.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout1.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout2.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout3.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout4.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout5.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout7.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout8.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout9.xml.rels  
  inflating: ppt/theme/theme1.xml    
 extracting: ppt/media/image1.png    
 extracting: ppt/media/hdphoto2.wdp  
 extracting: ppt/media/image7.png    
  inflating: ppt/vbaProject.bin      
 extracting: docProps/thumbnail.jpeg  
 extracting: ppt/media/hdphoto1.wdp  
 extracting: ppt/media/image5.png    
 extracting: ppt/media/image4.jpeg   
 extracting: ppt/media/image6.png    
 extracting: ppt/media/image3.png    
 extracting: ppt/media/image2.png    
  inflating: ppt/viewProps.xml       
  inflating: ppt/tableStyles.xml     
  inflating: ppt/presProps.xml       
  inflating: docProps/core.xml       
  inflating: docProps/app.xml
```

There's a lot of stuff in here. I figured macros would be written in VB, and sure enough there is a file `ppt/vbaProject.bin`. Strings shows what looked like the code for some functions:

![[Pasted image 20211119172932.png]]

I searched GitHub for some ways of opening VBA binary files. The [oletools](https://github.com/decalage2/oletools) repository stood out, so I installed it:

```bash
┌──(kali㉿kali)-[~/Documents/htb/uni-ctf]
└─$ sudo -H pip install -U oletools
```

The repo had instructions for analysing macros, using the `olevba` command:

![[Pasted image 20211119174505.png]]

I ran it against the binary:

```bash
┌──(kali㉿kali)-[~/Documents/htb/uni-ctf]
└─$ olevba reversing/upgrades/rev_upgrades/ppt/vbaProject.bin
olevba 0.60 on Python 3.9.7 - http://decalage.info/python/oletools
===============================================================================
FILE: reversing/upgrades/rev_upgrades/ppt/vbaProject.bin
Type: OLE
-------------------------------------------------------------------------------
VBA MACRO Module1.bas 
in file: reversing/upgrades/rev_upgrades/ppt/vbaProject.bin - OLE stream: 'VBA/Module1'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Private Function q(g) As String
q = ""
For Each I In g
q = q & Chr((I * 59 - 54) And 255)
Next I
End Function
Sub OnSlideShowPageChange()
j = Array(q(Array(245, 46, 46, 162, 245, 162, 254, 250, 33, 185, 33)), _
q(Array(215, 120, 237, 94, 33, 162, 241, 107, 33, 20, 81, 198, 162, 219, 159, 172, 94, 33, 172, 94)), _
q(Array(245, 46, 46, 162, 89, 159, 120, 33, 162, 254, 63, 206, 63)), _
q(Array(89, 159, 120, 33, 162, 11, 198, 237, 46, 33, 107)), _
q(Array(232, 33, 94, 94, 33, 120, 162, 254, 237, 94, 198, 33)))
g = Int((UBound(j) + 1) * Rnd)
With ActivePresentation.Slides(2).Shapes(2).TextFrame
.TextRange.Text = j(g)
End With
If StrComp(Environ$(q(Array(81, 107, 33, 120, 172, 85, 185, 33))), q(Array(154, 254, 232, 3, 171, 171, 16, 29, 111, 228, 232, 245, 111, 89, 158, 219, 24, 210, 111, 171, 172, 219, 210, 46, 197, 76, 167, 233)), vbBinaryCompare) = 0 Then
VBA.CreateObject(q(Array(215, 11, 59, 120, 237, 146, 94, 236, 11, 250, 33, 198, 198))).Run (q(Array(59, 185, 46, 236, 33, 42, 33, 162, 223, 219, 162, 107, 250, 81, 94, 46, 159, 55, 172, 162, 223, 11)))
End If
End Sub


-------------------------------------------------------------------------------
VBA MACRO Slide1.cls 
in file: reversing/upgrades/rev_upgrades/ppt/vbaProject.bin - OLE stream: 'VBA/Slide1'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Private Sub Label1_Click()

End Sub
+----------+--------------------+---------------------------------------------+
|Type      |Keyword             |Description                                  |
+----------+--------------------+---------------------------------------------+
|AutoExec  |Label1_Click        |Runs when the file is opened and ActiveX     |
|          |                    |objects trigger events                       |
|Suspicious|Environ             |May read system environment variables        |
|Suspicious|Run                 |May run an executable file or a system       |
|          |                    |command                                      |
|Suspicious|CreateObject        |May create an OLE object                     |
|Suspicious|Chr                 |May attempt to obfuscate specific strings    |
|          |                    |(use option --deobf to deobfuscate)          |
|Suspicious|Hex Strings         |Hex-encoded strings were detected, may be    |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
+----------+--------------------+---------------------------------------------+
```

### Function Analysis

The code contains a custom function `q` which is obfuscated. It looked to be some sort of custom encoding function, and I suspected the arrays being passed to it could be decoded. I replicated the code in an interactive Python terminal for the first few arrays:

```bash
┌──(kali㉿kali)-[~/Documents/htb/uni-ctf]
└─$ python                                                                                                                                               
Python 2.7.18 (default, Sep 24 2021, 09:39:51) 
[GCC 10.3.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> array = [245, 46, 46, 162, 245, 162, 254, 250, 33, 185, 33]
>>> array += [215, 120, 237, 94, 33, 162, 241, 107, 33, 20, 81, 198, 162, 219, 159, 172, 94, 33, 172, 94]
>>> array += [245, 46, 46, 162, 89, 159, 120, 33, 162, 254, 63, 206, 63]
>>> array += [89, 159, 120, 33, 162, 11, 198, 237, 46, 33, 107]
>>> array += [232, 33, 94, 94, 33, 120, 162, 254, 237, 94, 198, 33]
>>> decoded = [(I * 59 - 54) & 255 for I in array]
```

Here you can see it decodes the array into new integers, which I assume represent ASCII characters - nice!

![[Pasted image 20211119175244.png]]

We can turn these characters back into text with another list comprehension, using the `chr()` function - and now we start to get text back:

```bash
>>> ascii_decoded = [chr(c) for c in decoded]
>>> print(ascii_decoded)
['A', 'd', 'd', ' ', 'A', ' ', 'T', 'h', 'e', 'm', 'e', 'W', 'r', 'i', 't', 'e', ' ', 'U', 's', 'e', 'f', 'u', 'l', ' ', 'C', 'o', 'n', 't', 'e', 'n', 't', 'A', 'd', 'd', ' ', 'M', 'o', 'r', 'e', ' ', 'T', 'O', 'D', 'O', 'M', 'o', 'r', 'e', ' ', 'S', 'l', 'i', 'd', 'e', 's', 'B', 'e', 't', 't', 'e', 'r', ' ', 'T', 'i', 't', 'l', 'e']
```

Let's string it all together with the rest of the arrays. I used `join` to print out the full decoded arrays. Here's the complete Python:

```bash
┌──(kali㉿kali)-[~/Documents/htb/uni-ctf]
└─$ python                                                                                                                                               
Python 2.7.18 (default, Sep 24 2021, 09:39:51) 
[GCC 10.3.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> array = [245, 46, 46, 162, 245, 162, 254, 250, 33, 185, 33]
>>> array += [215, 120, 237, 94, 33, 162, 241, 107, 33, 20, 81, 198, 162, 219, 159, 172, 94, 33, 172, 94]
>>> array += [245, 46, 46, 162, 89, 159, 120, 33, 162, 254, 63, 206, 63]
>>> array += [89, 159, 120, 33, 162, 11, 198, 237, 46, 33, 107]
>>> array += [232, 33, 94, 94, 33, 120, 162, 254, 237, 94, 198, 33]
>>> decoded = [(I * 59 - 54) & 255 for I in array]
>>> print(decoded)
[65, 100, 100, 32, 65, 32, 84, 104, 101, 109, 101, 87, 114, 105, 116, 101, 32, 85, 115, 101, 102, 117, 108, 32, 67, 111, 110, 116, 101, 110, 116, 65, 100, 100, 32, 77, 111, 114, 101, 32, 84, 79, 68, 79, 77, 111, 114, 101, 32, 83, 108, 105, 100, 101, 115, 66, 101, 116, 116, 101, 114, 32, 84, 105, 116, 108, 101]
>>> ascii_decoded = [chr(c) for c in decoded]
>>> print(ascii_decoded)
['A', 'd', 'd', ' ', 'A', ' ', 'T', 'h', 'e', 'm', 'e', 'W', 'r', 'i', 't', 'e', ' ', 'U', 's', 'e', 'f', 'u', 'l', ' ', 'C', 'o', 'n', 't', 'e', 'n', 't', 'A', 'd', 'd', ' ', 'M', 'o', 'r', 'e', ' ', 'T', 'O', 'D', 'O', 'M', 'o', 'r', 'e', ' ', 'S', 'l', 'i', 'd', 'e', 's', 'B', 'e', 't', 't', 'e', 'r', ' ', 'T', 'i', 't', 'l', 'e']
>>> array += [81, 107, 33, 120, 172, 85, 185, 33]
>>> array += [154, 254, 232, 3, 171, 171, 16, 29, 111, 228, 232, 245, 111, 89, 158, 219, 24, 210, 111, 171, 172, 219, 210, 46, 197, 76, 167, 233]
>>> array += [215, 11, 59, 120, 237, 146, 94, 236, 11, 250, 33, 198, 198]
>>> array += [59, 185, 46, 236, 33, 42, 33, 162, 223, 219, 162, 107, 250, 81, 94, 46, 159, 55, 172, 162, 223, 11]
>>> decoded = [chr((I * 59 - 54) & 255) for I in array]
>>> print(''.join(decoded))
Add A ThemeWrite Useful ContentAdd More TODOMore SlidesBetter TitleusernameHTB{33zy_VBA_M4CR0_3nC0d1NG}WScript.Shellcmd.exe /C shutdown /S
```

And there's the flag! Nice!

![[Pasted image 20211119175626.png]]