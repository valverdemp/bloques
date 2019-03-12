I created a blog corpus of Spanish texts written by Japanese speakers who are learning Spanish. 

[blog_list.txt](https://github.com/valverdemp/bloques/blob/master/blog_list.txt) contains a list of 46 blogs written by 41 different learners.

[posts_list.txt](https://github.com/valverdemp/bloques/blob/master/posts_list.txt) contains the list of 2,669 URLs of blog posts, mainly from the Blogger and Wordpress domains, as of January 2015.

Here I explain the steps I followed to build the corpus, made up of 2,125 texts and 634,516 words. 

I uploaded the resulting corpus to the Sketch Engine for personal use. I cannot distribute it but I can "share" it with you if you are interested. You will not be able to download or modify but you can search it and download word lists, etc. You just have to let me know your username in Sketch Engine.

References:

Valverde, M.P. (2016), Japanese L1 speakers blogging in Spanish: motivations, topics and linguistic properties. In EPiC Series in Language and Linguistics. 8th International Conference on Corpus Linguistics, EasyChair, vol. 1, pp. 424-437, ISSN 2398-5283.

Valverde, M.P. (2017, to appear), Un corpus de blogs de aprendices japoneses de español (A blog corpus of Japanese learners of Spanish). Actas del XXVIII Congreso Internacional de ASELE.


# List of commands to create the blog corpus * UPDATE (2019): Nowadays you can compile a blog corpus with web scrapping software like import.io.


I did not use software like BootCat (with "Custom URLs" option) or TextSTAT, which allow you to build a corpus from a list of URLs, because I did not want to extract all the text from the web pages. I was interested only in the text in the main body, written by learners, and not the header and the comments section. Therefore, I needed to download first the HTML files and then use the HTML tags to locate and discard the header and comment sections. I followed these steps (you can skip step 1 and 2 if you use the files I make available).

### 1. Collect a list of blogs that are of your interest 

I collected a list of blogs that are written:

- In Spanish

- By speakers of Japanese as a first language who have studied or are studying Spanish as a foreign language

- Without a commercial purpose. They are "personal" blogs about hobbies, culture, opinion, etc.

I used Google and Blogspot search options and found 46 blogs written by 41 learners. The list is [blog_list.txt](https://github.com/valverdemp/bloques/blob/master/blog_list.txt).

### 2. Compile a list of URLs (one URL per line), one for every post

For every blog in the previous list, with a link-checker software (I used [Integrity](http://peacockmedia.software/mac/integrity/) for Mac), I made a list of the URLs I was interested in and saved the result in a .csv file. 

Link-checker software gives you the list of all links in a blog, so later I selected only the links to blog posts, by means of some regular expressions, e.g:

In blogspot:

```
awk -F "," '{print $2}' < yurikita.blogspot.csv | grep '^"http://yurikita.blogspot.jp/[0-9].*'|egrep -v '(archive|feed|trackback)'|tr -d '"' > yurikita.blogspot.txt
```

In Wordpress:

```
awk -F "," '{print $2}' < lqvelqp.wordpress.csv |grep '^"http://lqvelqp.wordpress.com/[0-9].*/[0-9].*/[0-9].*/[A-Za-z].*'|egrep -v '(twitter|facebook|tumblr|replyto|share|img|comment|-jpg|-png|attachment)' |tr -d '"' > lqvelqp.wordpress.txt
```

I did this for every blog, and saved each list in a .txt file. 
Once I had all the lists of posts, I concatenated them in one file:

```
cat *.txt > url_list.txt
```

The result is 2,669 URLs, that you can find in the file [posts_list.txt](https://github.com/valverdemp/bloques/blob/master/posts_list.txt).

### 3. Download the html files

From the directory where I wanted to save the corpus (I kept the posts_list.txt file in the URLS directory), I did:

```
wget -x -i ../URLS/posts_list.txt 
```

This downloaded the html files listed in posts_list.txt and created the same directory structure it found in the blog (blog title/year/month/day/post title). 

### 4. Reflect the directory structure in the filename

To flatten the directory structure, that is, to have all files in the same directory, and the names of the directory as part of the filename (blog title-year-month-day-post title, using "-" as a delimiter):

```
find . -mindepth 2 -type f -name '*' |
  perl -l000ne 'print $_;  s/\//-/g; s/^\.-/.\// and print' |
    xargs -0n2 mv 
```

Then I deleted the remaining empty directories:

```
find . -type d |xargs rm -rf
```

So I have all the html files of the corpus in one directory.

### 5. Make sure that everything is in UTF-8

```
file * | grep -v 'UTF-8'
```

I only found one blog in ISO-8859-15, so I changed the codification with iconv:

```
find . -name "pajarito*" -exec sh -c "iconv -f ISO-8859-15 -t UTF-8 {} > {}.utf8"  \; -exec mv "{}".utf8 "{}" \;
```

### 6. Select only the body text of the posts, and exclude the header and comment sections

There may be a better and more efficient way of doing this, but...

I made a list of tags that are used only at the beginning of the main body and a list of tags that appear only at the end of the main body, before the comments section.

Before (the html tags are shown slightly modified, as used later in regular expressions with sed):

```
BLOG TITLE            HTML TAG 
BLOGSPOT			<h3 class='post-title entry-title'
WORDPRESS			<div id=\"post-[0-9]+\"
pajarito			<h[0-9] class=\"entry-title\">
tarantos, etc		div class=\"post-[0-9]+
txirrindularijapon	<article id=\"post-[0-9]+\"
kimonoclub			<h3 class='entry-header'>
smileandfood y natadecocoblog <h3 class="post-title">
saborjapon			<div class=\"entry\">
```

Between body text and comments:

```
BLOG TITLE           HTML TAG 
BLOGSPOT 		<div class='post-footer'>
WORDPRESS		<div class=\"wpcnt\">
kimonoclub		<span class='post-timestamp'>
natadecodo		<p class=\"post-footer\">
saborjapon		<div class=\"post-ctr\">
```

And use *sed* to print only the text between any of the first tags and any of the second ones:

```
find . -name "*.html" -exec sed -E -i '.full' -n "/(<h3 class='post-title entry-title'|<div id=\"post-[0-9]+\"|<h[0-9] class=\"entry-title\">|div class=\"post-[0-9]+|<article id=\"post-[0-9]+\"|<h3 class='entry-header'>|<h3 class=\"post-title\">|<div class=\"entry\">)/,/(<div class='post-footer'>|<div class=\"wpcnt\">|<span class='post-timestamp'>|<p class=\"post-footer\">|<div class=\"post-ctr\">)/p" {} \; 
```

CAUTION: This command works in Mac, but you may have to modify it slightly if you use Linux.

* -E  is used to tell sed to interpret regular expressions as extended (modern) regular expressions rather than basic regular expressions (BRE's). This is necessary for using alternatives (|) in Mac.

* I used the backslash to scape double quotation marks ( " ) inside double quotation marks. Single quotation marks (') can be used without problem inside double quotes.

This file gave me an "illegal byte sequence"  message, so I deleted it:

```
rm tarantos.wordpress.com-2007-10-20-halloween-index.html
```

There are 14 files from which I could not get any text using the previous regular expressions:

```
ls -l|awk -F ' ' '{print $5, $9}' |grep -c '^0'
```

Finally, I deleted the .full files, which are a backup of the original files.

### 7. Convert html to text

In Mac (it's slow, it takes around 1 hour):

```
textutil -convert txt -inputencoding UTF-8 -encoding UTF-8 *
```

I also got some error messages, which mean the text file was not created for some reason.


### 8. Clean the texts

Move empty files to the directory EMPTY.

```
find . -type f -empty -exec mv {} EMPTY/ \;
```

Delete repeated lines and paragraphs written entirely in Japanese or English. I used this script.

```
find . -name '*txt' -exec sed -E -i '.bak' -f sedscr_exclude_mixed.repeated {} \;
```

Check which are the remaining repeated lines with, and add them to the previous script if it is necessary:

```
sort corpus_esp.txt|uniq -c |sort -n
```

### 9. Exclude some texts:

Move the texts with less than 30 words to the directory 0-30WORDS. Later delete the directory.

```
wc -w *txt | awk '$1<30{print $2}' | xargs -J {} mv {} 0-30WORDS/
```

I used ngramj to do automatic language detection for every file. For one file, for example:

```
java -jar cngram.jar -lang2 alalalajaponesa.blogspot.jp-2008-04-vamos-ver-las-flores-de-cerezo-ohanami.txt UTF-8
```

You get the following information: the text is 0,876 Spanish, 0,102 Portuguese...

```
speed: es:0,876 pt:0,102 ro:0,004 .. bg:0,000 |1,2E-1 |0,0E0 dt=1748
```

To do this for all the files in the directory at once:

```
for file in *.txt; do (echo "$file"; java -jar cngram.jar -lang2 $file UTF-8) ; done
```

I deleted these files:

```
rm -vfr `cat entradas_otras_lenguas.txt`
```

The result is 2125 texts and 634516 words.

