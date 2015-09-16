# List of commands to create the blog corpus BLOQUES

You can skip step 1 and 2.

###1. Collect a list of blogs that are of your interest. 

I collected a list of blogs that are written:
-In Spanish
-By speakers of Japanese as a first language who have studied or are studying Spanish as a foreign language
-Without a commercial purpose. They are "personal" blogs about hobbies, culture, opinion, etc.

I used Google and Blogspot search options and found 49 blogs written by 44 learners. The list is blogs_list.txt

###2. Compile a list of URLs (one URL per line), one for every blog post

For every blog in the previous list, with a link-checker software (I used [Integrity](http://peacockmedia.software/mac/integrity/) for Mac), I made a list of the URLs of every blog and saved the result in a .csv file. 

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
Once I had all the lists of blog posts, I concatenated them in one file:

```
cat *.txt > url_list.txt
```

The result is 2,683 URLs (that later resulted in around 2,000 texts). The list is in posts_list.txt.

###3. Download the html files

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

I only found one file in ISO-8859-15, so I changed the codification:

```
find . -name "pajarito*" -exec sh -c "iconv -f ISO-8859-15 -t UTF-8 {} > {}.utf8"  \; -exec mv "{}".utf8 "{}" \;
```

### 6. Select only the body text of the posts, and exclude the header and comment sections

There may be a better and more efficient way of doing this, but...

I made a list of tags that are used only at the beginning of the main body and a list of tags that only appear at the end of the main body, before the comments section.

Before (the html tags are shown slightly modified, as used later in regular expressions with sed):

```
BLOG TITLE  HTML TAG 
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
BLOG TITLE  HTML TAG 
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

There are also 15 files from which I could not get any text using the previous regular expressions:

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

TO BE CONTINUED
