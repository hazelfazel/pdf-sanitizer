# A simple PDF Sanitizer

We use this script to sanitize (aka flatten) given PDF files into images, then we convert the images back to PDF. Then we do an OCR on the imaged PDF. We call the final PDF sanitized. The script runs on a secured, read-only live linux box. So even if a PDF exploits something, an attacker only has access to a read-only live-linux box. You can check out a demo at <a href="https://letsopen.it" title="letsopen.it">letsopen.it</a>

Besides to PDFs we are also able to convert DOCX, XLSX, PPTX into PDFs and then process them the same way using libre office or a licensed tools under NDA. If you are interested in more details, feel free and contact me.

## What is it good for?

For many business users it is important to open document files. Examples are press department, human resources, client communication, etc. Business customers often cannot ignore provided document files just because of a suspicion or bad feeling. In some business areas documents shall be opened by regulations. Even though the document is nonsense or in worst case could contain malware. This is risky in terms of IT security. Typical online document analysis tools do not respect privacy, especially GDPR. So I build a system that could be run as an appliance in a demilitarized zone, fully under control and easily GDPR conformant. It receives PDFs (and other document files) and converts them into PDF sanitized the way presented here.

## Tools required

Ensure you have installed ImageMagick, Poppler-Utils, qpdf and ocrmypdf.

Make sure you have Ghostscript ≥9.24:

<pre>gs --version</pre>

<pre>
sudo apt-get install --no-install-recommends imagemagick poppler-utils qpdf ocrmypdf tesseract-ocr-deu tesseract-ocr-eng
</pre>

You may edit /etc/ImageMagick-<INSTALLED-VERSION>/policy.xml 

You should set:

	policy domain="resource" name="memory" value="512MiB"
	policy domain="resource" name="disk" value="4GiB"
	policy domain="coder" rights="read|write" pattern="PDF"
	
## Script

<pre>
#!/bin/bash

cd $(dirname $0)

if test -f "convert.pdf"; then
	echo $(date +"%T")
	convert -density 150 -background white -alpha background -alpha off -strip "convert.pdf" -quality 50 "converted-%05d.JPG"
	echo $(date +"%T")
	mogrify -format pdf -- *.JPG
	pdfunite converted*.pdf final-sample.pdf
	echo $(date +"%T")
	ocrmypdf -l deu+eng --output-type pdf final-sample.pdf final-ocr-sample.pdf
	echo $(date +"%T")
	rm -f converted*
	rm -f convert.pdf
fi
</pre>
