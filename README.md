# List of commands to create the blog corpus BLOQUES

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

The result is 2,683 URLs (that later resulted in around 2,000 texts). The list is in posts_list.txt
