---
ctf: picoctf
competition: false
categories: forensics
tools: file, unzip, grep, oleid, olevba
url: https://play.picoctf.org/practice/challenge/130
captured: 
flag: 
---

# MacroHard WeakEdge

> I've hidden a flag in this file. Can you find it? [Forensics is fun.pptm](https://mercury.picoctf.net/static/9a7436948cc502e9cacf5bc84d2cccb5/Forensics is fun.pptm)

First things first, find out what kind of file you have using `file`.

```shell
$ file Forensics\ is\ fun.pptm 
Forensics is fun.pptm: Microsoft PowerPoint 2007+
```

Ah, a Microsoft Office document!

> Broadly speaking, there are two generations of Office file format: the OLE formats (file extensions like RTF, DOC, XLS, PPT), and the "Office Open XML" formats (file extensions that include DOCX, XLSX, PPTX). Both formats are structured, compound file binary formats that enable Linked or Embedded content (Objects). OOXML files are actually zip file containers (see the section above on archive files), meaning that one of the easiest ways to check for hidden data is to simply `unzip` the document [source](https://trailofbits.github.io/ctf/forensics/#office-file-analysis)

```shell
$ unzip Forensics\ is\ fun.pptm
Archive:  Forensics is fun.pptm
  inflating: [Content_Types].xml     
  inflating: _rels/.rels             
  inflating: ppt/presentation.xml    
  inflating: ppt/slides/_rels/slide46.xml.rels  
  inflating: ppt/slides/slide1.xml   
  inflating: ppt/slides/slide2.xml   
  ... snip ...
```

We can try `grep`-ing for the flag recursively through all the files.

```shell
$ grep -r "flag"  
ppt/slides/slide37.xml:...<a:t>Not the flag</a:t>
```

Funnily enough, we have a hit! ...or more like a _hint_. Maybe we're not suppose to find the flag in the static part of the file. The other thing about offices files is a thing called VBA (Visual Basic Application) macros. 

> You can include Visual Basic for Applications (VBA) code or run COM add-ins only in a macro-enabled document, worksheet, or presentation. You can create a macro-enabled file by saving the documents with a .docm or .dotm extension in Word; an .xlsm, .xltm, or .xlam extension in Excel; or a .pptm, .potm, .ppam, or .ppsm extension in PowerPoint. [source](https://learn.microsoft.com/en-us/office/vba/library-reference/concepts/security-notes-for-microsoft-office-solution-developers)

These VBA macros can do almost anything on a Windows system and provide attacks a handy vector for things like ransomware and creating back doors.

>A typical VBA macro in an Office document, on Windows, will download a PowerShell script to %TEMP% and attempt to execute it, in which case you now have a PowerShell script analysis task too. But malicious VBA macros are rarely complicated, since VBA is [typically just used as a jumping-off platform to bootstrap code execution](https://www.lastline.com/labsblog/party-like-its-1999-comeback-of-vba-malware-downloaders-part-3/). [source](https://trailofbits.github.io/ctf/forensics/#office-file-analysis)


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

Ha! Another false flag!

