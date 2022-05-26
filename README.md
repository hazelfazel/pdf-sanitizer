# A simple PDF Sanitizer

I use this script to sanitize (aka flatten) given PDF files into images, then I convert the images back to PDF. Then I do an OCR on the imaged PDF. I call the final PDF sanitized. The script runs on a secured, read-only live linux box running in memory only. So even if a PDF exploits something, an attacker will end in my limited read-only live-linux box. This means that the risk is very low.

Next to PDFs I am also able to convert DOCX, XLSX, PPTX into PDFs and then process them the same way. As this part uses licensed tools under NDA I cannot share all details here. If you are interested in more information, feel free and contact me.

## What is it good for?

For many business users it is important to open document files. Examples are press department, human resources, client communication, etc. Business customers often cannot ignore provided document files just because of a suspicion or bad feeling. In some business areas documents shall be opened by regulations. Even though the document is nonsense or in worst case could contain malware. This is risky in terms of IT security. Typical online document analysis tools do not respect privacy, especially GDPR. So I build a system that could be run as an appliance in a demilitarized zone, fully under control and easily GDPR conformant. It receives PDFs (and other document files) and converts them into PDF sanitized the way presented here.

## Tools required

Ensure you have installed ImageMagick, Poppler-Utils, qpdf and ocrmypdf

<pre>
sudo apt-get install --no-install-recommends imagemagick poppler-utils qpdf ocrmypdf tesseract-ocr-deu tesseract-ocr-eng
</pre>

You may edit /etc/ImageMagick-<INSTALLED-VERSION>/policy.xml 

You should set:

	<policy domain="resource" name="memory" value="512MiB"/>
	<policy domain="resource" name="disk" value="4GiB"/>
	
	<policy domain="coder" rights="read|write" pattern="PDF"/>
	
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
