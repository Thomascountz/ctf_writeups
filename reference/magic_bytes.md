The traditional heuristic for identifying filetypes on UNIX is libmagic, which is a library for identifying so-called "magic numbers" or "magic bytes," the unique identifying marker bytes in filetype headers. The libmagic libary is the basis for the file command.

$ file screenshot.png 
screenshot.png: PNG image data, 1920 x 1080, 8-bit/color RGBA, non-interlaced

Keep in mind that heuristics, and tools that employ them, can be easily fooled.