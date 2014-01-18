---
layout: post
title: Web Scraping
description: "Scraping a web series using Python"
modified: 2014-01-18
category: hacking
tags: [web scraping, beautifulsoup, python, worm, wildbow]
comments: true
share: true
---

I recently came across a web-series called [Worm by Wildbow](http://parahumans.wordpress.com/). It is fairly popular and astoundingly long (more than 1.6 million words which is more than all of the [ASOIAF](http://en.wikipedia.org/wiki/A_Song_of_Ice_and_Fire) combined). Naturally, I wanted to read it but reading it on the laptop is tiresome and exhausting and I could not find any e-book version of it online to read on my kindle. This is where some awesome and easy to use python modules for scraping came in handy. Following is a simple tutorial to web scraping in python

###Modules Used

* [Beautiful Soup](http://www.crummy.com/software/BeautifulSoup/) - Install it by running `apt-get install python-bs4` or `easy_install beautifulsoup4`
* urllib2


###Using urllib2 to fetch a link
Its pretty simple.Just do
{% highlight python %}
url = 'ninjaducks.in'
url = urllib2.quote(url,safe=":/")
response = urllib2.urlopen(url)
html_doc = response.read()
{% endhighlight %}

`urllib2.quote()` function in the second line is used to convert all the special characters in the string to compatible format.

###The Splendid Beautiful Soup

Beautiful Soup is a library used for navigating, searching and modifying DOM of the HTML received while scraping. To use it, include it in your document and convert your HTML document to Beautiful Soup object by

{% highlight python %}
from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc)
{% endhighlight %}


Now Beautiful Soup provides functions on this object to navigate the DOM.

{% highlight python %}
#find element with an ID. Equivalent to getElementById in javascript
soup.find(id="myUniqueID")

#find all elelments which have a certain class
soup.find_all(class_='myClass'})
or
soup.find_all(attrs={'class': 'myClass'}

#find the first link in the document
soup.a

#get the href attribute of the a tag
soup.a['href']

#find all div tags in the document
soup.find_all('div')

#get the text inside  the first div
soup.div.get_text()
{% endhighlight %}

These functions help us to get the content of the page and the link to the next page.

### Writing the fetched content to a file

The string given by `get_text()` function of beautiful soup is encoded in UTF-8 format (to support characters from other languages and some special characters too). Writing it directly to the file will give errors. We need to call encode function on the string before writing it to the file.

{% highlight python %}
fil = open('worm.html', 'a')
str  = 'text to be written in file'
fil.write(str.encode('utf_8'))
{% endhighlight %}

###Converting the content to e-book

[Calibre](http://calibre-ebook.com/) is a software for managing e-books. It also lets us convert e-books from one format to other. We can import HTML files in it (it shows html files as zip) and convert them to azw or mobi which are supported by kindle. Scraping and storing the relevant html from the site also prevents the formatting of the text from getting lost. So, I stored all the HTML related to the book in a file and 


##Full code

{% gist 8491226 worm.py %}


Download the ebook version of Worm from [here](http://goo.gl/XnGyqX)