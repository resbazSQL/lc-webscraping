---
title: "Manually scrape data using browser extensions"
teaching: 45
exercises: 20
questions:
- "How can I get started scraping data off the web?"
- "How can I use XPath to more accurately select what data to scrape?"
objectives:
- "Introduce the Chrome Scraper extension."
- "Practice scraping data that is well structured."
- "Use XPath queries to refine what needs to be scraped when data is less structured."
keypoints:
- "Data that is relatively well structured (in a table) is relatively easily to scrape."
- "More often than not, web scraping tools need to be told what to scrape."
- "XPath can be used to define what information to scrape, and how to structure it."
- "More advanced data cleaning operations are best done in a subsequent step."
---

# Using the Scraper Chrome extension

Now we are finally ready to do some web scraping. Let's go back to the list of
[UK House of Commons members](https://www.parliament.uk/mps-lords-and-offices/mps/). 

We are interested in downloading this list to a spreadsheet, with columns for names and
constituencies. Do do so, we will use the Scraper extension in the Chrome browser
(refer to the [Setup](setup/) section for help installing these tools).

## Scrape similar

With the extension installed, we can select the first row of the House of Commons members
list, do a right click and choose "Scrape similar" from the contextual menu:

![Screenshot of the Scraper contextual menu]({{ page.root }}/fig/scraper-contextmenu.png)

Alternatively, the "Scrape similar" option can also be accessed from the Scraper extension
icon:

![Screenshot of the Scraper menu]({{ page.root }}/fig/scraper-menu.png)

Either operation will bring up the Scraper window:

![Screenshot of the Scraper main window]({{ page.root }}/fig/scraper-ukparl-01.png)

## Xpath and the Document Object Model

Note the "Selector" on the left column. It should read:

~~~
//tbody/tr[td]
~~~

![A good output of the scraper]({{ page.root }}/fig/goodManualScrape.png)


What happens if you don't highlight the full row and right click scrape similiar? We see something like:

~~~
//tr[2]/td
~~~
![A bad output of the scraper]({{ page.root }}/fig/badManualScrape.png)

The difference in these two "XPath Queries" is where they're telling the scrape extension to look within the "Document object model" of the webpage. We will be exploring these in a few minutes, but right now, we need to get this data to a usable "spreadsheet" format.

## Cleaning scraped Data

One characteristic of scraped data is that it is very seldom formatted for clean computer consumption. If we choose "Copy to Clipboard" and we paste the data copied from Scraper into a text document, we see something like this:

~~~
Name  Constituency
A back to top
                                 Abbott, Ms Diane                                 (Labour)                              Hackney North and Stoke Newington
                                 Abrahams, Debbie                                 (Labour)                              Oldham East and Saddleworth
~~~
{: .output}

This is because there are a lot of unnecessary white spaces in the HTML that's behind that table, which
are being captured by Scraper. We can however tweak the XPath column selectors to take advantage of the
`normalize-space` XPath function:

~~~
normalize-space(*[1])
normalize-space(*[2])
~~~

![Screenshot of the Scraper window showing the Column selectors]({{ page.root }}/fig/scraper-ukparl-02.png)

We now need to tell Scraper to scrape the data again by using our new selectors, this is done by clicking
on the "Scrape" button. The preview will not noticeably change, but if we now copy again the results
and paste them in our text editor, we should see

~~~
Name  Constituency
A back to top
Abbott, Ms Diane (Labour) Hackney North and Stoke Newington
Abrahams, Debbie (Labour) Oldham East and Saddleworth
Adams, Nigel (Conservative) Selby and Ainsty
~~~
{: .output}

which is a bit cleaner.

> ## Scrape the list of Australian MPPs
> Use Scraper to export the list of the first twelve [Members of the Australian Parliment](https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0). [https://perma.cc/8ATF-RT3Q](https://perma.cc/8ATF-RT3Q).
> and try exporting the results in your favourite spreadsheet or data analysis
> software. 
> BBS FIXME
> Once you have done that, try adding a third column containing the district of each member of parliment.
>
> Tips:
> 
> We first need to find the html tag that "holds" each row. 
> 
> * To add another column in Scraper, use the little green "+" icon in the columns list.
> * Look at the source code and try out XPath queries in the console until you find what
>   you are looking for.
> * The syntax to select the value of an attribute of the type `<element attribute="value">`
>   is `element/@attribute`.
> * The `concat()` XPath function can be use to concatenate things.
>
> > ## Solution
> > 
> > Add a third column with the XPath query
> > 
> > ~~~
> > *[1]/a/@href
> > ~~~
> > {: .source}
> >
> > ![Screenshot of the Scraper window on the Ontario MPP page]({{ page.root }}/fig/scraper-ontparl-01.png)
> > 
> > This extracts the URLs, but as luck would have it, those URLs are relative to the list
> > page (i.e. they are missing `http://www.ontla.on.ca/web/members/`). We can use the
> > `concat()` XPath function to construct the full URLs:
> >
> > ~~~
> > concat('http://www.ontla.on.ca/web/members/',*[1]/a/@href)
> > ~~~
> > {: .source}
> >
> > ![Screenshot of the Scraper window on the Ontario MPP page]({{ page.root }}/fig/scraper-ontparl-02.png)
> >
> {: .solution}
{: .challenge}
