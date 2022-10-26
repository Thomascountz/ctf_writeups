---
ctf: picoctf
competition: false
categories: forensics
tools: file, unzip
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

