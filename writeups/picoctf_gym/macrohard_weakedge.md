---
ctf: picoctf
competition: false
categories: [forensics]
tools: [file, unzip, grep, oleid, olevba, base64]
url: https://play.picoctf.org/practice/challenge/130
captured: 2022-10-26
flag: picoCTF{D1d_u_kn0w_ppts_r_z1p5}
summary: The Windows document was unzipped to reveal a hidden file containing a base64 encoded flag.
---

# MacroHard WeakEdge

> I've hidden a flag in this file. Can you find it?

First things first, find out what kind of file you have using `file`.

```shell
$ file Forensics\ is\ fun.pptm 
Forensics is fun.pptm: Microsoft PowerPoint 2007+
```

Ah, a Microsoft Office document!

> Broadly speaking, there are two generations of Office file format: the OLE formats (file extensions like RTF, DOC, XLS, PPT), and the "Office Open XML" formats (file extensions that include DOCX, XLSX, PPTX). Both formats are structured, compound file binary formats that enable Linked or Embedded content (Objects). OOXML files are actually zip file containers (see the section above on archive files), meaning that one of the easiest ways to check for hidden data is to simply `unzip` the document. [^1]

Let's use `zipinfo` so that we don't actually end up with the files and directories in our filesystem.

```shell
$ zipinfo Forensics\ is\ fun.pptm 
Archive:  Forensics is fun.pptm
Zip file size: 100093 bytes, number of entries: 153
-rw----     2.0 fat    10660 b- defS 80-Jan-01 00:00 [Content_Types].xml
-rw----     2.0 fat      738 b- defS 80-Jan-01 00:00 _rels/.rels
-rw----     2.0 fat     5197 b- defS 80-Jan-01 00:00 ppt/presentation.xml
-rw----     2.0 fat      311 b- defS 80-Jan-01 00:00 ppt/slides/_rels/slide46.xml.rels
-rw----     2.0 fat     1740 b- defS 80-Jan-01 00:00 ppt/slides/slide1.xml
...snip...
-rw----     2.0 fat     8783 b- defS 80-Jan-01 00:00 ppt/_rels/presentation.xml.rels
-rw----     2.0 fat      311 b- defS 80-Jan-01 00:00 ppt/slides/_rels/slide1.xml.rels
...snip...
-rw----     2.0 fat    13875 b- defS 80-Jan-01 00:00 ppt/slideMasters/slideMaster1.xml
...snip...
-rw----     2.0 fat     1991 b- defS 80-Jan-01 00:00 ppt/slideMasters/_rels/slideMaster1.xml.rels
...snip...
-rw----     2.0 fat     8399 b- defS 80-Jan-01 00:00 ppt/theme/theme1.xml
-rw----     2.0 fat     2278 b- stor 80-Jan-01 00:00 docProps/thumbnail.jpeg
-rw----     2.0 fat     7168 b- defS 80-Jan-01 00:00 ppt/vbaProject.bin
-rw----     2.0 fat      818 b- defS 80-Jan-01 00:00 ppt/presProps.xml
-rw----     2.0 fat      811 b- defS 80-Jan-01 00:00 ppt/viewProps.xml
-rw----     2.0 fat      182 b- defS 80-Jan-01 00:00 ppt/tableStyles.xml
-rw----     2.0 fat      666 b- defS 80-Jan-01 00:00 docProps/core.xml
-rw----     2.0 fat     3784 b- defS 80-Jan-01 00:00 docProps/app.xml
-rw-ah-     2.0 fat       99 t- defN 20-Oct-23 14:31 ppt/slideMasters/hidden
153 files, 237329 bytes uncompressed, 77909 bytes compressed:  67.2%
```

After inspecting, we can call `unzip` and the try `grep`-ing for the flag recursively through all the files.

```shell
$ grep -r "flag"  
ppt/slides/slide37.xml:...<a:t>Not the flag</a:t>
```

Funnily enough, we have a hit! ...or more like a _hint_. Maybe we're not suppose to find the flag in the static part of the file. The other thing about offices files is a thing called VBA (Visual Basic Application) macros. 

> You can include Visual Basic for Applications (VBA) code or run COM add-ins only in a macro-enabled document, worksheet, or presentation. You can create a macro-enabled file by saving the documents with a .docm or .dotm extension in Word; an .xlsm, .xltm, or .xlam extension in Excel; or a .pptm, .potm, .ppam, or .ppsm extension in PowerPoint. [^2] 

These VBA macros can do almost anything on a Windows system and provide attacks a handy vector for things like ransomware and creating back doors.

>A typical VBA macro in an Office document, on Windows, will download a PowerShell script to %TEMP% and attempt to execute it, in which case you now have a PowerShell script analysis task too. But malicious VBA macros are rarely complicated, since VBA is [typically just used as a jumping-off platform to bootstrap code execution](https://www.lastline.com/labsblog/party-like-its-1999-comeback-of-vba-malware-downloaders-part-3/). [^3]

Let's use `oleid` to see if there are any VBA macros embedded in our file.

```shell
$ oleid Forensics\ is\ fun.pptm 

Filename: Forensics is fun.pptm
--------------------+--------------------+----------+--------------------------
Indicator           |Value               |Risk      |Description               
--------------------+--------------------+----------+--------------------------
File format         |MSPointpoint 2007+  |info      |                          
                    |Macro-Enabled       |          |                          
                    |Presentation (.pptm)|          |                          
--------------------+--------------------+----------+--------------------------
Container format    |OpenXML             |info      |Container type            
--------------------+--------------------+----------+--------------------------
Encrypted           |False               |none      |The file is not encrypted 
--------------------+--------------------+----------+--------------------------
VBA Macros          |Yes                 |Medium    |This file contains VBA    
                    |                    |          |macros. No suspicious     
                    |                    |          |keyword was found. Use    
                    |                    |          |olevba and mraptor for    
                    |                    |          |more info.                
--------------------+--------------------+----------+--------------------------
XLM Macros          |No                  |none      |This file does not contain
                    |                    |          |Excel 4/XLM macros.       
--------------------+--------------------+----------+--------------------------
External            |0                   |none      |External relationships    
Relationships       |                    |          |such as remote templates, 
                    |                    |          |remote OLE objects, etc   
--------------------+--------------------+----------+--------------------------
```

Sure enough, a VBA macro has been detected, although it notes that "no suspicious keyword was found."

Let's take a look with `olevba`.

```shell
$ olevba Forensics\ is\ fun.pptm 

===============================================================================
FILE: Forensics is fun.pptm
Type: OpenXML
-------------------------------------------------------------------------------
VBA MACRO Module1.bas 
in file: ppt/vbaProject.bin - OLE stream: 'VBA/Module1'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Sub not_flag()
    Dim not_flag As String
    not_flag = "sorry_but_this_isn't_it"
End Sub
No suspicious keyword or IOC found.
```

Ha! Another false flag...

Looking back up at the files that were `unzip`-ed, I just noticed one called `ppt/slideMasters/hidden`?

```shell
$ ls -l ppt/slideMasters/
total 20
drwxrwxr-x 2 thomascountz-picoctf thomascountz-picoctf    35 Oct 26 21:34 _rels
-rw-rw-r-- 1 thomascountz-picoctf thomascountz-picoctf    99 Oct 23  2020 hidden
-rw-rw-r-- 1 thomascountz-picoctf thomascountz-picoctf 13875 Jan  1  1980 slideMaster1.xml

$ cat ppt/slideMasters/hidden
Z m x h Z z o g c G l j b 0 N U R n t E M W R f d V 9 r b j B 3 X 3 B w d H N f c l 9 6 M X A 1 f Q
```

That looks like an encoded something! Perhaps base64?

```bash
$ cat ppt/slideMasters/hidden | sed 's/ //g' |  base64 -d 
flag: picoCTF{D1d_u_kn0w_ppts_r_z1p5}
```

It is! How do we recognize base64 encoded strings? One way is use a tool like [CyberChef's "Magic" operation](https://gchq.github.io/CyberChef/#recipe=Magic(3,false,false,'')). Another way is to be familiar with the constraints of base64 encoding and recognize its ubiquity in CTFs:

>Encoded data will always have the following characteristic:
> -   The length of a Base64-encoded string is always a multiple of 4
> -   Only these characters are used by the encryption: “A” to “Z”, “a” to “z”, “0” to “9”, “+” and “/”
> -   The end of a string can be padded up to two times using the “=”-character (this character is allowed in the end only)
> [^4] 

[^1]: https://trailofbits.github.io/ctf/forensics/#office-file-analysis
[^2]: https://learn.microsoft.com/en-us/office/vba/library-reference/concepts/security-notes-for-microsoft-office-solution-developers
[^3]: https://trailofbits.github.io/ctf/forensics/#office-file-analysis
[^4]: https://www.hannesholst.com/blog/how-to-identify-a-base64-encoded-string/
