---
title: Reduce PDF Size on Linux with Ghostscript
date: 2025-02-23
summary: With gs you can reduce the PDF size on Linux.
categories: [Linux, PDF, Ghostscript, Compression, Command Line]
---

Have you ever had a PDF file that was unnecessarily large? Whether it's for emailing, archiving, or simply saving space, reducing the size of a PDF can be incredibly useful. On Linux, one of the most effective tools for this task is `gs`, short for [Ghostscript](https://www.ghostscript.com/).

`gs` is a versatile interpreter for PostScript and PDF files. It can do a lot, but we're focusing on its ability to optimize PDF files. The key is to use the right combination of options.

The general command structure is:

```bash
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/screen -dNOPAUSE -dQUIET -dBATCH -sOutputFile=output.pdf input.pdf
```

Let's break down the most important parts:

* `-sDEVICE=pdfwrite`: Specifies that we want to create a PDF file.
* `-dCompatibilityLevel=1.4`: Sets the PDF compatibility level. 1.4 is a good balance for older viewers while still allowing some modern features. You can experiment with higher levels (like 1.5, 1.6, etc.) if needed.
* `-dPDFSETTINGS=/screen`:  This is the crucial option! It controls the level of compression applied. Here are a few common options:
    * `/screen`: Lowest quality, smallest file size. Good for on-screen viewing.
    * `/ebook`: Medium quality, good for e-readers and general use.
    * `/printer`: High quality, suitable for printing.
    * `/prepress`: Highest quality, intended for professional printing.
* `-dNOPAUSE -dQUIET -dBATCH`: These options tell Ghostscript to run non-interactively, suppress informational messages, and exit after processing.
* `-sOutputFile=output.pdf`: Specifies the name of the output PDF file.
* `input.pdf`: The name of the PDF file you want to compress.

## Examples

### Compressing for Emailing

Let's say you have a PDF named `original.pdf` that's too large to email. You can use the following command to create a smaller version:

```bash
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/screen -dNOPAUSE -dQUIET -dBATCH -sOutputFile=report.pdf original.pdf
```

This will create a new file named `report.pdf` with a significantly reduced file size, suitable for emailing.

### Compressing for Archiving

For archiving documents where print quality isn't paramount but readability is desired, use the `/ebook` setting:

```bash
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/ebook -dNOPAUSE -dQUIET -dBATCH -sOutputFile=archive.pdf original.pdf
```

This balances quality and size, making it a good choice for long-term storage.
