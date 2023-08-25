---
title: 'How to OCR a PDF on Linux'
description: "Step-by-step tutorial on how how to OCR a PDF document on Linux."
preview_image: /assets/images/blog/2023/how-to-ocr-pdfs-in-linux/article-header.png
section: blog
author:
   - Yasoob Khalid
   - Teja Tatimatla
date: 2023-05-09 9:00 UTC
category: Tutorials
products: PSPDFKit Processor 
tags: Linux, Processor, OCR
published: true
secret: false
---

In this tutorial, we'll discuss how to perform OCR on Linux using an open source solution and also cover how to use PSPDFKit Processor for more [advanced OCR use cases](https://pspdfkit.com/guides/processor/ocr/overview/).

The open source library we'll be using is [OCRmyPDF](https://ocrmypdf.readthedocs.io/en/latest/). This is a multi-platform tool for running OCR on PDF files based on the open-source OCR engine, Tesseract.

## How to OCR a PDF on Linux Using an Open Source Library

### Why Not Use Tesseract Directly?

[OCRmyPDF](https://ocrmypdf.readthedocs.io/en/latest/) is a wrapper around Tesseract that does some pre-processing on PDF files before running OCR on them. This pre-processing includes deskewing, noise removal, and cleaning up the PDF file to ensure that the OCR engine can read the text accurately. OCRmyPDF also does some post-processing to ensure that the output is consistent and error-free. You can use Tesseract directly but you will miss out on these benefits provided by OCRmyPDF. 

### Installing OCRmyPDF

You can install OCRmyPDF using the following command on Ubuntu or Debian-based systems:

```
$ sudo apt-get install ocrmypdf
```

For Fedora you can use the following command:

```
$ dnf install ocrmypdf
```

Sometimes the APT version might not be the latest one so you can install OCRmyPDF directly from PIP too:

```
$ pip install --user ocrmypdf
```

Just keep in mind that the PIP method will not install some non-Python dependencies of OCRmyPDF. These dependencies include:

- Python 3.8 or newer
- Ghostscript 9.50 or newer
- Tesseract 4.1.1 or newer
- jbig2enc 0.29 or newer
- pngquant 2.5 or newer
- unpaper 6.1

### Basic Usage

To use OCRmyPDF, you can simply run the following command, replacing `input.pdf` with the path to the PDF file you want to OCR, and `output.pdf` with the path where you want to save the OCR'd PDF:

```bash
$ ocrmypdf input.pdf output.pdf
```

This will result in a PDF/A output file with an OCR layer. PDF/A is a subset of the PDF standard that prohibits features that are not suitable for long-term archiving. This includes JS in PDFs, font linking, and encryption. You can ask OCRmyPDF to output a standard PDF via this command:

```bash
$ ocrmypdf --output-type pdf input.pdf output.pdf
```

You can even perform OCR only on certain pages:

```bash
$ ocrmypdf --pages 2,3,13-17 input.pdf output.pdf
```

### OCR in a Different Language Than English

By default, OCRmyPDF assumes that the document is in English. If the language is different, the OCR quality will be considerably poor. You need to explicitly pass in the language in these cases:

```bash
$ ocrmypdf -l rus russian_doc.pdf russian_doc_ocr.pdf
```

If the document is multi-lingual, you can pass in multiple languages:

```
$ ocrmypdf -l rus+eng russian_doc.pdf russian_doc_ocr.pdf
```

Tesseract (the OCR engine used by OCRmyPDF under the hood) supports quite a few different languages. You can take a look at the [Tesseract documentation](https://github.com/tesseract-ocr/tesseract/blob/main/doc/tesseract.1.asc#languages) to figure out if it supports your required language. 

You might be required to install additional language packs before you can use them with OCRmyPDF. Follow these [instructions](https://ocrmypdf.readthedocs.io/en/latest/languages.html#lang-packs) to figure out how to do so.

### Image Processing

As mentioned earlier, OCRmyPDF can perform some image processing on each page of the PDF if required. It supports multiple options for this purpose. According to [the official docs](https://ocrmypdf.readthedocs.io/en/latest/cookbook.html#image-processing), there are 5 different options:

- `--rotate-pages` attempts to determine the correct orientation for each page and rotates the page if necessary.
- `--remove-background` attempts to detect and remove a noisy background from grayscale or color images. Monochrome images are ignored. This should not be used on documents that contain color photos as it may remove them.
- `--deskew` will correct pages that were scanned at a skewed angle by rotating them back into place.
- `--clean` uses [unpaper](https://www.flameeyes.eu/projects/unpaper) to clean up pages before OCR, but does not alter the final output. This makes it less likely that OCR will try to find text in background noise.
- `--clean-final` uses unpaper to clean up pages before OCR and inserts the page into the final output. You will want to review each page to ensure that unpaper did not remove something important.

No matter which order you pass in these options, OCRmyPDF will always apply them in this order:

```
rotate -> remove background -> deskew -> clean
```

### File Optimization

OCRmyPDF by default optimizes the output PDF for "fast web view". This linearizes the PDF file and stores all references in the PDF file in the same order in which they will be viewed by the user. This slightly increases the file size as well but can be disabled by passing in `--optimize 0` or `-O0`.

At the default optimization level `-O1`, OCRmyPDF also does some lossless image optimization using JBIG2 encoder. You can disable this optimization by passing in `-O0` or you can enable more aggressive lossy optimization by passing in `-O2` or `-O3`.

### Batch Processing PDF Files

By default, OCRmyPDF uses all available cores while processing PDF files. You can limit this by using the `-j` or `--jobs` option. This limits the number of concurrent threads used:

```bash
$ ocrmypdf -j 4 input.pdf output.pdf
```

The authors of the program have also conveniently created a [`watcher.py` file](https://github.com/ocrmypdf/OCRmyPDF/blob/master/misc/watcher.py) that you can use to watch folders and perform OCR on any new PDF file. You might need to update the contents of the watcher file to suit your specific needs. Because this file has some additional dependencies, you might need to install `ocrmypdf` using the `watcher` tag:

```bash
$ pip install ocrmypdf[watcher]
```

You can then run the watcher like this:

```bash
$ env OCR_INPUT_DIRECTORY=./input-pdfs \
    OCR_OUTPUT_DIRECTORY=./output-pdfs \
    python3 watcher.py
```

This will OCR any new PDF files that are placed inside `input-pdfs` folder and place the resulting PDF in the `output-pdfs` folder. Do note that this will not process any files that were already in the `input-pdfs` folder before the watcher was run.

## How to OCR a PDF on Linux Using PSPDFKit Processor

PSPDFKit Processor offers a powerful solution for batch processing, manipulating, and editing PDF documents. It allows you to automate complex OCR workflows and scale 

- Easy to deploy - no database or storage required.

- Processor doesn’t store any document data or information internally so you remain in complete control of your documents at all times.

- Simple to use — upload a document with a set of operations and get a modified document back

- Horizontally scalable — adding more nodes linearly increases your processing capacity.

- Works on PDF files, Office documents, and images.


### Requirements to Get Started

To get started, you'll need:

* [Docker Engine](https://docs.docker.com/engine/install/#server)
* [Docker Compose](https://docs.docker.com/compose/install/#install-compose-on-linux-systems)
* [Curl](https://curl.se)

> By default, curl comes bundled with most Linux distributions. You can check if it’s installed on your machine by running `curl --version`

### Setting up PSPDFKit Processor

Start the processor by running:

```cmd
sudo docker run --rm -t -p 5000:5000 pspdfkit/processor:2023.7.0
```

> When run for the first time, docker will pull the image from the repository. Depending on your internet connection speed, this might take a while.

After successful execution, you should see the following on your terminal.
![Image showing the terminal out-put on successful execution](/assets/images/blog/2023/how-to-ocr-pdfs-in-linux/starting-pspdfkit-processor.png)

## Build API Reference for PDF Processor

The processor's `/build` API takes a building blocks like approach to construct a PDF document from multiple [parts](https://pspdfkit.com/guides/processor/api/build-api-reference/#parts). Furthermore, each of the parts may come from multiple sources, such as an existing PDF, a black page, an HTML page, or an image file.

With the `/build` API you can apply one or more [actions](https://pspdfkit.com/guides/processor/api/build-api-reference/#actions) including OCR to each part. This is done by supplying an `instructions` JSON.

For example, If you wish to perform OCR on specific pages, say pages 2 and 3 (indexes 1 and 2), add the first page (index 0) to the `parts` field of the `instructions` JSON.

```JSON
{
  "parts": [
    {
      "file": "document",
      "pages": {
        "start": 0,
        "end": 0
      }
    }
  ]
}
```

Then, add the pages 2 and 3. Since you'd like to apply OCR to these pages, you'll add the actions' field with the type `ocr` to this part.

```JSON
{
  "parts": [
    {
      "file": "document",
      "pages": {
        "start": 0,
        "end": 0
      }
    },
    {
      "file": "document",
      "pages": {
        "start": 1,
        "end": 2
      },
      "actions": [
        {
          "type": "ocr",
          "language": "english"
        }
      ]
    },
  ]
}
```

Now, add the remaining pages in the document to the `parts` field.

```JSON
{
  "parts": [
    {
      "file": "document",
      "pages": {
        "start": 0,
        "end": 0
      }
    },
    {
      "file": "document",
      "pages": {
        "start": 1,
        "end": 2
      },
      "actions": [
        {
          "type": "ocr",
          "language": "english"
        }
      ]
    },
    {
      "file": "document",
      "pages": {
        "start": 3,
        "end": 7
      }
    }
  ]
}
```

However, if you want to apply `OCR` on the entire document, the `instructions` JSON will look like the following:

```JSON
{
  "parts": [
    {
      "file": "document"
    }
  ],
  "actions": [
    {
      "type": "ocr",
      "language": "english"
    }
  ]
}
```

Actions that are common to all the pages in the input PDF are described in the `actions` field outside the `parts` field.

You'll send the `instructions` JSON and the path to the input PDF (the PDF you want to perform OCR on) as a `multipart/form-data` request (using the -F option in the curl command) to the Processor's `/build` endpoint.

### Running OCR on All Pages

To perform `OCR` on all pages of a PDF document, make sure that the PSPDFKit processor is running and execute the following command:

```cmd
curl -X POST http://localhost:5000/build \
  -F document=@/path/to/example-document.pdf \
  -F instructions='{
  "parts": [
    {
      "file": "document"
    }
  ],
  "actions": [
    {
      "type": "ocr",
      "language": "english"
    }
  ]
}' \
  -o result.pdf
```
> Replace '/path/to/example-document.pdf' with the actual path to your document.

The -o option will save the resulting document as `result.pdf` in the current working directory.

Before `OCR`:

![Image showing the search result in a document before OCR is applied](/assets/images/blog/2023/how-to-ocr-pdfs-in-linux/brfore-ocr.png)

After `OCR`:

![Image showing the search result in a document after OCR is applied](/assets/images/blog/2023/how-to-ocr-pdfs-in-linux/after-ocr.png)

### Running OCR on Specific Pages of a Document

To perform `OCR` on Specific Pages of a Document, run:

```cmd
curl -X POST http://localhost:5000/build \
  -F document=@/path/to/example-document.pdf \
  -F instructions='{
  "parts": [
    {
      "file": "document",
      "pages": {
        "start": 0,
        "end": 0
      }
    },
    {
      "file": "document",
      "pages": {
        "start": 1,
        "end": 2
      },
      "actions": [
        {
          "type": "ocr",
          "language": "english"
        }
      ]
    },
    {
      "file": "document",
      "pages": {
        "start": 3,
        "end": 7
      }
    }
  ]
}' \
  -o result.pdf
```

### Running OCR on a Document from a URL

To specify the path of a document for OCR using a URL, use the `url` field:

```cmd
curl -X POST http://localhost:5000/build \
  -F instructions='{
  "parts": [
    {
      "file": {
        "url": "https://pspdfkit.com/downloads/examples/paper.pdf"
      }
    }
  ],
  "actions": [
    {
      "type": "ocr",
      "language": "english"
    }
  ]
}' \
  -o result.pdf
```


## Conclusion

In this article, you learned how to OCR PDFs on Linux using an open source library OCRmyPDF, and how you can cover more advanced use cases by using PSPDFKit Processor.

To learn more you can check out our Processor [getting started guides](https://pspdfkit.com/getting-started/processor/) or [reach out](https://pspdfkit.com/sales/form/) to our team to get more information. 