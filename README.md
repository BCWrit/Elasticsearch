# Archiviz

Archiviz is a collection of three plug-ins for Omeka that allow users to digitize text documents (OCR), identify important people, places, and things in them (TM or Tag Management), and then visualize and navigate their collection through an interactive knowledge graph showing where these important entities intersect (Archiviz, a modded version of Harvard's Elasticsearch wrapper for Omeka).

Users install Archiviz in a standard Omeka Classic server, with a few additional necessary dependencies. If you've set up an Omeka server, you should be perfectly capable of executing these additional steps. If you had help setting up the server, you might want to pull in your helper for this.

## Setting up the server

When you're ready to start, you'll need to ssh into your Omeka server using the syntax ```ssh [yourusername]@[serverURLorIP]```, where you should put your corresponding information in the brackets. You'll be prompted for your password, which you should enter. Once on the system, you should type '''sudo bash''' to give yourself admin rights to the shell. You'll need these to install the dependencies. 

From here, paste the following command into the prompt: ```apt install apache2 mysql-server imagemagick php-mysql php-imagick unzip net-tools libapache2-mod-php php-xml php-mysql php-mbstring php-zip php-soap php-curl php-gd php-imap php-common php-dev libmcrypt-dev php-pear php-exif tesseract-ocr-eng```. This installs a variety of pretty standard web dev and sys admin packages necessary for getting Archiviz working. 

You'll need to configure your new Apache modules: ```a2enmod rewrite```. And also initiate your MySQL database: ```systemctl enable mysql``` and then ```mysql_secure_installation```. You'll be prompted to create a password. Mark this down as you'll need it to initiate the Omeka MySQL user account in the next section.

Tesseract for OCR functionality: ```apt install tesseract-ocr-eng```.

And finally sPaCy for named entity recognition: ```pip install nltk spacy``` and ```python3 -m spacy download en_core_web_sm```.

## Imagemagick and PHP set-up

Next, we need to change a few values on the server to devote more resources to the additional packages we've installed. To set up ImageMagick, type ```nano /etc/ImageMagick-7/policy.xml```. This will open up a simple text editor where you need to change two lines. Comment out (type a "#" before the line) ```<policy domain="coder" rights="read | write" pattern="PDF" />``` and change ```<policy domain="resource" name="disk" value="1GiB"/>``` to ```<policy domain="resource" name="disk" value="8GiB"/>```. For PHP, you'll type ```nano /etc/php/[version]/apache2/php.ini``` (Again, you'll need to provide the correct version number in the bracketed portion: you may need to navigate to that filepath to check the directory name.) You'll change the value of ```upload\max\_filesize = 2M``` to desired size (suggested: 30M) and the value of ```post \_max_size = 8M```. Finally, ```nano /application/config/config.ini``` and uncomment and change the value of ";upload.maxFileSize" to "10M". These changes increase the size of files Omeka, PHP, and ImageMagick will allow: you may or may not need it, but these values worked for us on our 10,000 document test collection.

## Omeka set-up

Navigate in a web browser to the url of your Omeka server and log in. Use the menus to navigate to "Admin -> Settings -> General -> ImageMagick Directory". In this menu, you'll type in "/usr/bin/convert/" into the field for "ImageMagick Directory". In "Admin -> Settings -> API", check the box for "Enable API".

## Last step: Install the plug-ins
Back in the shell, ```cd /var/www/[omeka directory]/plugins```. Again, you'll need to check and insert the name of the Omeka directory on your server: this will change as new versions are released. Type ```wget https://github.com/BCWrit/Archiviz```, which will download this repo to your server. Then type ```unzip [name of zipfile]``` to unzip the plug-ins.
