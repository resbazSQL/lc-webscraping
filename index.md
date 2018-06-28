---
layout: lesson
root: .
---

Web scraping is the process of extracting data from websites. Some data is presented in a format that is easy to collect and use, for example the comma-separated value (CSV) format specifies a simple structure for presenting data in a file. CSV files may be downloaded and imported into a spreadsheet, statistical analysis application or parsed by a script for custom transformation or analysis. Often however, even though publicly available, data is not conveniently accessible for reuse. 
For example it might be contained in a PDF, or a table on a website, or spread across multiple web pages.

There are a variety of ways to _scrape_ a website to extract information for reuse.
In its simplest form, this can be achieved by copying and pasting snippets from a web page, but this is impractical when there is a large amount of data to extract, or when it is spread over many pages. Instead, specialized tools and techniques exist to automate the process. You can define what sites to visit, what information to look for, when to stop, and when to continue. These tools will follow hyperlinks and extract data recursively from multiple pages. You can also automate web scraping to run at regular intervals and capture changes in the data.


> ## Prerequisites
>
> As webscraping is a technique to extract data from web pages, it requires some understanding of
> the technologies that are used to display information on the web. 
> This lesson therefore assumes that learners will have some familiarity with [HTML](https://en.wikipedia.org/wiki/HTML)
> and the [Document Object Model](https://en.wikipedia.org/wiki/Document_Object_Model) (DOM).
>  
> * For chapters 2 and 3 of this lesson, all you need is [Google Chrome](https://www.google.com/chrome/) and the [Scraper extension](https://chrome.google.com/webstore/detail/scraper/mbigbapnjcgaffohmbkdlecaccepngjd).
> 
> * Chapters 4 and 5 of this lesson introduce the use of specialized libraries to scrape websites by writing
> custom computer programs and will require some familiarity with the 
> [Python programming language](https://swcarpentry.github.io/python-novice-inflammation/)
> and [object-oriented programming](https://en.wikipedia.org/wiki/Object-oriented_programming).
>
{: .prereq}

## Software requirements

Refer to the [Setup](setup/) section to install the required software to follow along this lesson.

> ## Under development
>
> Please note that the contents of this lesson are still being actively developed. Any feedback is
> appreciated, please do not hesitate to contact [the author](mailto:tom@timtom.ch) or contribute
> to the lesson by [forking it on GitHub](https://github.com/timtomch/library-webscraping/).
>
{: .callout}
