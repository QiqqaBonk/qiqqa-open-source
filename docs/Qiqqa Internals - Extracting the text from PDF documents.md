# The Qiqqa OCR \[Background\] Process

Before we dive in, there's one important question to ask (when considering storage size/costs and Qiqqa backwards compatibility):


## Given a PDF, *what* does Qiqqa store on disk?

It does not matter *how* Qiqqa obtained the incoming PDF document, be it by "watch folder" directory scanning, website sniffer download, drag&drop or other means to import: all incoming PDFs are processed the same way.

> Some **metadata** bits may be different: a source URL may be saved on Sniffer download or alike, but that's about it.

- The incoming **original PDF** is copied to the Qiqqa Library **document store**, which is located in the `<LibraryID>/documents/` directory tree.

  The PDF **content** is hashed (using a [SHA1 derivative](https://github.com/jimmejardine/qiqqa-open-source/blob/0b015c923e965ba61e3f6b51218ca509fcd6cabb/Utilities/Files/StreamFingerprint.cs#L14)) to produce a unique identifier for this particular PDF **content**. That hash is used throughout Qiqqa for indexing *and* is to *name* the cached version of the incoming PDF, using a simple yet effective distribution scheme to help NTFS/filesystem performance for large libraries: the first character of the hash is also used as a *subdirectory* name. 
  
  Example path for a PDF file stored in the `Guest` Qiqqa Library:

  ```
    base/Guest/documents/D/DA7B8FDA82E6D7465ADC7590EEC0C914E955C5B8.pdf
  ```

- The extracted text is saved in a Qiqqa-global store at `base/ocr/` using a similar filesystem performance scheme as for the PDF  file itself.

  
  Example paths for the OCR output cached for the same PDF file as shown above:

  ```
    base/ocr/DA/DA7B8FDA82E6D7465ADC7590EEC0C914E955C5B8.pagecount.0.txt
    base/ocr/DA/DA7B8FDA82E6D7465ADC7590EEC0C914E955C5B8.text.4.txt
    base/ocr/DA/DA7B8FDA82E6D7465ADC7590EEC0C914E955C5B8.textgroup.001_to_020.txt
    base/ocr/DA/DA7B8FDA82E6D7465ADC7590EEC0C914E955C5B8.textgroup.021_to_040.txt
  ```
  
  > Note that in this example, we apparently had a PDF which had its page 4 OCRed using `tesseract` (a.k.a. the **SINGLE** process), while the other 20+ pages got extracted using `mupdf` (a.k.a. the **GROUP** process): apparently the given PDF was a text-based PDF which *possibly* an empty page or a full-page graphic without embedded text on page 4.
  >
  > See the process description below for more info.
  
  The **TEXT DATA** stored in these 'ocr' files uses a custom text format, where each word is listed on a separate line and accompanied by a set of coordinates describing the rectangle of its location within the page.
  
  Example OCR text file snippet:
  
  ```
# Generated by: QiqqaOCR.
# Version: 3
# List source: PDFText
# System culture: en-US
@PAGE: 1

0.62114,0.04798,0.11382,0.01641:USOO695.2431B1

0.12683,0.08586,0.02602,0.02904:(12)

0.15935,0.08586,0.08455,0.02904:United

0.25366,0.08586,0.07480,0.02904:States

0.33984,0.08586,0.07967,0.02904:Patent

0.52683,0.08586,0.02602,0.02904:(10)

0.55935,0.08586,0.05528,0.02904:Patent

0.62114,0.08586,0.03415,0.02904:No.:

0.69593,0.08586,0.03089,0.02904:US

0.73333,0.08586,0.09106,0.02904:6,952,431

0.83252,0.08586,0.02602,0.02904:B1

0.15772,0.10732,0.04553,0.02399:Dally

0.20813,0.10732,0.01626,0.02399:et

0.22927,0.10732,0.02276,0.02399:al.

0.52683,0.10732,0.02602,0.02399:(45)

0.55935,0.10732,0.03902,0.02399:Date

0.60325,0.10732,0.02114,0.02399:of

0.62764,0.10732,0.05854,0.02399:Patent:

0.75772,0.10732,0.03740,0.02399:Oct.

0.79837,0.10732,0.01626,0.02399:4,

0.81789,0.10732,0.03902,0.02399:2005

0.12683,0.14899,0.02602,0.01641:(54)

0.16585,0.14899,0.05528,0.01641:CLOCK

0.22439,0.14899,0.10569,0.01641:MULTIPLYING

0.33333,0.14899,0.11707,0.01641:DELAY-LOCKED

0.53821,0.14899,0.05366,0.01641:6,037,812

0.59675,0.14899,0.01138,0.01641:A

0.63577,0.14899,0.03740,0.01641:3/2000

0.68293,0.14899,0.03902,0.01641:Gaudet

0.72683,0.14899,0.08455,0.01641:.......................

0.81463,0.14899,0.04390,0.01641:327/116

0.16748,0.16035,0.04228,0.01641:LOOP

0.21301,0.16035,0.03089,0.01641:FOR

0.24878,0.16035,0.03902,0.01641:DATA

0.29106,0.16035,0.14146,0.01641:COMMUNICATIONS

0.53821,0.16035,0.05366,0.01641:6,043,717

0.59675,0.16035,0.01138,0.01641:A

0.63577,0.16035,0.03740,0.01641:3/2000

0.68293,0.16035,0.02764,0.01641:Kurd

0.71545,0.16035,0.09919,0.01641:...........................

0.82114,0.16035,0.01463,0.01641:33
  ```
  
  As you can already see, a 'word' here is not always in accordance of the human purview of the meaning of 'word', e.g. the 'word' `...........................` at the end of the snippet there.
  
  Qiqqa [applies a few filters to this data](https://github.com/jimmejardine/qiqqa-open-source/blob/1ef3403788d2b2d5efcc08dc244a60d1694f5453/Qiqqa/DocumentLibrary/DocumentLibraryIndex/LibraryIndex.cs#L629-L638) before it is injected into the `Lucene` search index database.


## The Process


### Fetching the PDF


### The Qiqqa OCR Background Process

- Once the background task gets around to it, the PDF is OCRed if this has not happened yet. 

  > This is detected by checking whether the expected OCR data for page 1 is available.
  >
  > Yes, that last statement right there is not entirely accurate either. The correct(er) answer is: *it depends*: several conditions exist (e.g. when the document is viewed by the user in a Qiqqa panel), when *all pages* of the document are requested and any of them missing will (re)trigger the OCR process. (See all the invocations of [the `GetOCRText()` method](https://github.com/jimmejardine/qiqqa-open-source/blob/1ef3403788d2b2d5efcc08dc244a60d1694f5453/Qiqqa/Documents/PDF/PDFRendering/PDFRenderer.cs#L98) in the Qiqqa source code.)

  - First, Qiqqa attempts to [extract text from the PDF without OCR-ing it, using the `mupdf` tool](https://github.com/jimmejardine/qiqqa-open-source/blob/1ef3403788d2b2d5efcc08dc244a60d1694f5453/QiqqaOCR/TextExtractEngine.cs#L178): this should deliver for all PDFs which are not 'page image based'.

    > However, Qiqqa *will* trigger a OCR action for those pages of the PDF which do not produce any text this way.
    >
    > In actual practice, this means many text-based PDFs will have an OCR job running for them as there's an empty page, or one with only some graphics, or a title page, which did not deliver any text by way of `mupdf`.


### The Lucene Text SearchIndex Update Process

