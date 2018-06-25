---
title: "Selecting content on a web page with XPath"
teaching: 30
exercises: 15
questions:
- "How can I select a specific element on web page?"
- "What is XPath and how can I use it?"
objectives:
- "Introduce XPath queries"
- "Explain the structure of an XML or HTML document"
- "Explain how to view the underlying HTML content of a web page in a browser"
- "Explain how to run XPath queries in a browser"
- "Introduce the XPath syntax"
- "Use the XPath syntax to select elements on this web page"
keypoints:
- "XML and HTML are markup languages. They provide structure to documents."
- "XML and HTML documents are made out of nodes, which form a hierarchy."
- "The hierarchy of nodes inside a document is called the node tree."
- "Relationships between nodes are: parent, child, sibling."
- "XPath queries are constructed as paths going up or down the node tree."
- "XPath queries can be run in the browser using the `$x()` function."
---

~~~


We recognize that Scraper has generated XPath queries that corresponds to the data we had
selected upon calling it. The Selector (highlighted in red in the above screenshot)
 has been set to `//tbody/tr[td]` which selects
all the rows of the table, delimiting the data we want to extract.

In fact, we can try out that query using the technique that we learned in the previous
section by typing the following in the browser console:

~~~
$x("//tbody/tr[td]")
~~~
{: .source}

returns something like

~~~
<- Array [672]
~~~
{: .output}

which we can explore in the console to make sure this is the right data.

Scraper also recognized that there were two columns in that table, and has accordingly
created two such columns (highlighted in blue in the screenshot), 
each with its own XPath selector, `*[1]` and `*[2]`.

To understand what this means, we have to remember that XPath queries are relative to the
current context node. The context node has been set by the Selector query above, so
those queries are relative to the array of `tr` elements that has been selected.

We can replicate their effect by trying out 

~~~
$x("//tbody/tr[td]/*[1]")
~~~
{: .source}

in the console. This should select only the first column of the table. The same goes for the
second column.

But in this case, we don't need to fiddle with the XPath queries too much, as Scraper was able to deduce
them for us, and we can use the export functions to either create a Google Spreadsheet with the
results, or copy them into the clipboard in Tab Separated Values (TSV) format for pasting into
a text document, a spreadsheet or Open Refine.

There is one bit of data cleanup we might want to do, though. If we paste the data copied from Scraper
into a text document, we see something like this:

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

> ## Scrape the list of Ontario MPPs
> Use Scraper to export the list of [current members of the Ontario Legislative Assembly](https://www.ola.org/en/members/current)
> and try exporting the results in your favourite spreadsheet or data analysis
> software.
>
> Once you have done that, try adding a third column containing the URLs that are underneath
> the names of the MPPs and that are leading to the detail page for each parliamentarian.
>
> Tips:
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



## Custom XPath queries

Sometimes, however, we do have to do a bit of work to get Scraper to select the data elements
that we are interested in.

Going back to the example of the Canadian Parliament we saw in the introduction,
there is a page on the same website that [lists the mailing addresses](http://www.parl.gc.ca/Parliamentarians/en/members/addresses) of all
parliamentarians. We are interested in scraping those addresses.

If we select the addresses for the first MP and try the "Scrape similar" function...

![Screenshot of the Scraper context menu being used on an address block]({{ page.root }}/fig/scraper-canparl-01.png)

Scraper produces this:

![Screenshot of the Scraper window trying to scrape addresses]({{ page.root }}/fig/scraper-canparl-02.png)

which does a nice job separating the address elements, but what if instead we want a table of
the addresses of all MPs? Selecting multiple addresses instead does not help. Remember what we said
about computers not being smart about structuring information? This is a good example. We humans
know what the different blocks of texts on the screen mean, but the computer will need some help from
us to make sense of it.

We need to tell Scraper what exactly to scrape, using XPath.

If we look at the HTML source code of that page, we see that individual MPs are all within `ul`
elements:

~~~
(...)
<ul>
   <li><h3>Aboultaif, Ziad</h3></li>
   <li>
      <span class="addresstype">Hill Office</span>
      <span>Telephone:</span>
      <span>613-992-0946</span>
      <span>Fax:</span>            
      <span>613-992-0973</span>
   </li>
   <li>
         <ul>       
            <li><span class="addresstype">Constituency Office(s)</span></li>
            <li>                            
               <span>8119 - 160 Avenue (Main Office)</span>
               <span>Suite 204A</span>
               <span>Edmonton, Alberta</span>
               <span>T5Z 0G3</span>
               <span>Telephone:</span> <span>780-822-1540</span>
               <span>Fax:</span> <span>780-822-1544</span>                                    
               <span class="spacer"></span>
            </li>                         
      </ul>
   </li>                                   
</ul>   
(...)
~~~
{: .output}

So let's try changing the Selector XPath in Scraper to

~~~
//body/div[1]/div/ul
~~~
{: .source}

and hit "Scrape". We get something that is closer to what we want, with one line per MP, but
the addresses are still all in one block of unstructured text:

![Screenshot of the Scraper window trying to scrape addresses]({{ page.root }}/fig/scraper-canparl-03.png)

Looking closer at the HTML source, we see that name and addresses are separated by `li` elements
within those `ul` elements. So let's add a few columns based on those elements:

~~~
./li[1] -> Name
./li[2] -> Hill Office
./li[3] -> Constituency
~~~
{: .source}

This produces the following result:

![Screenshot of the Scraper window scraping addresses]({{ page.root }}/fig/scraper-canparl-04.png)

The addresses are still one big block of text each, but at least we now have a table for all MPs
and the addresses are separated.

> ## Scrape the Canadian MPs' phone numbers
> Keep working on the example above to add a column for the Hill Office phone number
> and fax number for each MP.
>
>
> > ## Solution
> > 
> > Add columns with the XPath query
> > 
> > ~~~
> > ./li[2]/span[3] -> Hill Office Phone
> > ./li[2]/span[5] -> Hill Office Fax
> > ~~~
> > {: .source}
> >
> > ![Screenshot of the Scraper window on scraping MP phone numbers]({{ page.root }}/fig/scraper-canparl-05.png)
> >
> {: .solution}
{: .challenge}

~~~


Before we delve into web scraping proper, we will first spend some time introducing
some of the techniques that are required to indicate exactly what should be
extracted from the web pages we aim to scrape.

The material in this section was adapted from the [XPath and XQuery Tutorial](https://github.com/code4libtoronto/2016-07-28-librarycarpentrylessons/blob/master/xpath-xquery/lesson.md)
written by [Kim Pham](https://github.com/kimpham54) ([@tolloid](https://twitter.com/tolloid))
for the July 2016 [Library Carpentry workshop](https://code4libtoronto.github.io/2016-07-28-librarycarpentry/) in Toronto.

# Introduction
XPath (which stands for XML Path Language) is an _expression language_ used to specify parts of an XML document.
XPath is rarely used on its own, rather it is used within software and languages that are aimed at manipulating
XML documents, such as XSLT, XQuery or the web scraping tools that will be introduced later in this lesson.
XPath can also be used in documents with a structure that is similar to XML, like HTML.

## Markup Languages
XML and HTML are _markup languages_. This means that they use a set of tags or rules to organise and provide
information about the data they contain. This structure helps to automate processing, editing, formatting,
displaying, printing, etc. that information.

XML documents stores data in plain text format. This provides a software- and hardware-independent way of storing,
transporting, and sharing data. XML format is an open format, meant to be software agnostic. You can
open an XML document in any text editor and the data it contains will be shown as it is meant to be represented.
This allows for exchange between incompatible systems and easier conversion of data.


> ## XML and HTML
>
> Note that HTML and XML have a very similar structure, which is why XPath can be used almost interchangeably to
> navigate both HTML and XML documents. In fact, starting with HTML5, HTML documents are fully-formed XML documents.
> In a sense, HTML is like a particular dialect of XML.
>
{: .callout}

XML document follows basic syntax rules:

* An XML document is structured using _nodes_, which include element nodes, attribute nodes and text nodes
* XML element nodes must have an opening and closing tag, e.g. `<catfood>` opening tag and `</catfood>` closing tag
* XML tags are case sensitive, e.g. `<catfood>` does not equal `<catFood>`
* XML elements must be properly nested:

```
<catfood>
  <manufacturer>Purina</manufacturer>
    <address> 12 Cat Way, Boise, Idaho, 21341</address>
  <date>2019-10-01</date>
</catfood>
```
* Text nodes (data) are contained inside the opening and closing tags
* XML attribute nodes contain values that must be quoted, e.g.
``` <catfood type="basic"></catfood> ```

# XPath Expressions

XPath is written using expressions, and when these expressions are evaluated on XML documents they return an object
containing the node(s) that you aim to select. Contrary to a flat text document, XML data is structured, as it is
organized in nodes and subnodes.
Therefore, when using XPath, we are not querying raw text or data values like we would so using Regular Expressions,
for example. Instead, XPath makes use of the fact that XML documents are structured and instead navigates through the
node structure to select the data that we are looking for.

XPath is typically used to select and compare nodes, not edit them. To manipulate or edit nodes, another language such
as XQuery would be used instead.

> ## XPath assumes _structured_ data
>
> We can think of using XPath as similar to search a library catalogue using the advanced search function.
> In a catalogue, we can take advantage of the fact that bibliographic information has been properly structured
> in the database by specifying which metadata fields we want to query. For example, if we are looking for books
> about Shakespeare but not those written by him, we can use the advanced search function to look for that name
> in the "title" field only.
>
> Contrary to Regular Expressions, this means that we don't have to know in advance what the data we are looking for
> looks like, we just need to know in which node(s) (or fields) it resides.
>
{: .callout}

Now let's start using XPath.

## Navigating through the HTML node tree using XPath

A popular way to represent the structure of an XML or HTML document is the _node tree_:

![HTML Node Tree](http://www.w3schools.com/js/pic_htmltree.gif)

In an HTML document, everything is a node:

* The entire document is a document node
* Every HTML element is an element node
* The text inside HTML elements are text nodes

The nodes in such a tree have a hierarchical relationship to each other. We use the terms _parent_, _child_ and
_sibling_ to describe these relationships:

* In a node tree, the top node is called the *root* (or *root node*)
* Every node has exactly one *parent*, except the root (which has no parent)
* A node can have zero, one or several *children*
* *Siblings* are nodes with the same parent
* The sequence of connections from node to node is called a *path*

![Node relationships](http://www.w3schools.com/js/pic_navigate.gif)

Paths in XPath are defined using slashes (`/`) to separate the steps in a node connection sequence, much like
URLs or Unix directories.

In XPath, all expressions are evaluated based on a *context node*. The context node is the node in which a path
starts from. The default context is the root node, indicated by a single slash (/), as in the example above.

The most useful path expressions are listed below:

| Expression   | Description |
|-----------------|:-------------|
| ```nodename```| Select all nodes with the name "nodename"   |
| ```/```  | A beginning single slash indicates a select from the root node, subsequent slashes indicate selecting a child node from current node  |
| ```//``` | Select direct and indirect child nodes in the document from the current node - this gives us the ability to "skip levels" |
| ```.```       | Select the current context node   |
|```..```  | Select the parent of the context node|
|```@```  | Select attributes of the context node|
|```[@attribute = 'value']```   |Select nodes with a particular attribute value|
|`text()`| Select the text content of a node|
| &#124;|Pipe chains expressions and brings back results from either expression, think of a set union |


## Navigating through a webpage with XPath using a browser console

We will use the HTML code that describes this very page you are reading as an example. By default, a web browser
interprets the HTML code to determine what markup to apply to the various elements of a document, and the code is
invisible. To make the underlying code visible, all browsers have a function to display the raw HTML content of
a web page.

> ## Display the source of this page
> Using your favourite browser, display the HTML source code of this page.
>
> Tip: in most browsers, all you have to do is do a right-click anywhere on the page and select the "View Page Source"
> option ("Show Page Source" in Safari).
>
> Another tab should open with the raw HTML that makes this page. See if you can locate its various elements, and
> this challenge box in particular.
>
{: .challenge}


> ## Using the Safari browser
>
> If you are using Safari, you must first turn on the "Develop" menu in order to view the page source, and use the
> functions that we will use later in this section. To do so, navigate to Safari > Preferences and in the Advanced tab
> select the "Show Develop in menu bar" option. Note: In recent versions of Safari you must first turn on the "Develop" menu (in Preferences) and then navigate to `Develop > Show Javascript Console` and then click on the "Console" tab.
>
{: .callout}

The HTML structure of the page you are currently reading looks something like this (most text and elements have
been removed for clarity):

~~~
<!doctype html>
<html lang="en">
  <head>
    (...)
    <title>{{page.title}}</title>
  </head>
  <body>
	 (...)
  </body>
</html>
~~~
{: .output}

We can see from the source code that the title of this page is in a `title` element that is itself inside the
`head` element, which is itself inside an `html` element that contains the entire content of the page.

Say we wanted to tell a web scraper to look for the title of this page, we would use this information to indicate the
_path_ the scraper would need to follow at it navigates through the HTML content of the page to reach the `title`
element. XPath allows us to do that.

We can run XPath queries directly from within all major modern browsers, by enabling the built-in JavaScript console.

> ## Display the console in your browser
>
> * In Firefox, use to the *Tools > Web Developer > Web Console* menu item.
> * In Chrome, use the *View > Developer > JavaScript Console* menu item.
> * In Safari, use the *Develop > Show Error Console* menu item. If your Safari browser doesn't have a Develop menu,
>   you must first enable this option in the Preferences, see above.
>
{: .callout}

Here is how the console looks like in the Firefox browser:

![JavaScript console in Firefox]({{ page.root }}/fig/firefox-console.png)

For now, don't worry too much about error messages if you see any in the console when you open it. The console
should display a _prompt_ with a `> ` character (`>>` in Firefox) inviting you to type commands.

The syntax to run an XPath query within the JavaScript console is `$x("XPATH_QUERY")`, for example:

~~~
$x("/html/head/title/text()")
~~~
{: .source}

This should return something similar to

~~~
<- Array [ #text "{{page.title}}" ]
~~~
{: .output}

The output can vary slightly based on the browser you are using. For example in Chrome, you have to "open" the
return object by clicking on it in order to view its contents.

Let's look closer at the XPath query used in the example above: `/html/head/title/text()`. The first `/` indicates
the _root_ of the document. With that query, we told the browser to

|-----------------|:-------------|
| `/`| Start at the root of the document... |
| `html/`| ... navigate to the `html` node ... |
| `head/`| ... then to the `head` node  that's inside it... |
| `title/`| ... then to the `title` node that's inside it... |
| `text()`| and select the text node contained in that element |

Using this syntax, XPath thus allows us to determine the exact _path_ to a node.

> ## Select the "Introduction" title
> Write an XPath query that selects the "Introduction" title above and try running it in the console.
>
> Tip: if a query returns multiple elements, the syntax `element[1]` can be used. Note that
> XPath uses one-based indexing, therefore the first element has index 1, the second has index 2 etc.
>
> > ## Solution
> >
> > ~~~
> > $x("/html/body/div/article/h1[1]")
> > ~~~
> > {: .source}
> >
> > should produce something similar to
> >
> > ~~~
> > <- Array [ <h1#introduction> ]
> > ~~~
> > {: .output}
> >
> {: .solution}
{: .challenge}

Before we look into other
ways to reach a specific HTML node using XPath, let's start by looking closer at how nodes are arranged
within a document and what their relationships with each others are.


For example, to select all the `blockquote` nodes of this page, we can write

~~~
$x("/html/body/div/article/blockquote")
~~~
{: .source}

This produces an array of objects:

~~~
<- Array [ <blockquote.objectives>, <blockquote.callout>, <blockquote.callout>, <blockquote.challenge>, <blockquote.callout>, <blockquote.callout>, <blockquote.challenge>, <blockquote.challenge>, <blockquote.challenge>, <blockquote.keypoints> ]
~~~
{: .output}

This selects all the `blockquote` elements that are under `html/body/div`. If we want instead to select _all_
`blockquote` elements in this document, we can use the `//` syntax instead:

~~~
$x("//blockquote")
~~~
{: .source}

This produces a longer array of objects:

~~~
<- Array [ <blockquote.objectives>, <blockquote.callout>, <blockquote.callout>, <blockquote.challenge>, <blockquote.callout>, <blockquote.callout>, <blockquote.challenge>, <blockquote.solution>, <blockquote.challenge>, <blockquote.solution>, 3 more… ]
~~~
{: .output}

> ## Why is the second array longer?
> If you look closely into the array that is returned by the `$x("//blockquote")` query above,
> you should see that it contains objects like `<blockquote.solution>` that were not
> included in the results of the first query. Why is this so?
>
> Tip: Look at the source code and see how the challenges and solutions elements are
> organised.
>
{: .challenge}

We can use the `class` attribute of certain elements to filter down results. For example, looking
at the list of `blockquote` elements returned by the previous query, and by looking at this page's
source, we can see that the blockquote elements on this page are of different classes
(challenge, solution, callout, etc.).

To refine the above query to get all the `blockquote` elements of the `challenge` class, we can type

~~~
$x("//blockquote[@class='challenge']")
~~~
{: .source}

which returns

~~~
Array [ <blockquote.challenge>, <blockquote.challenge>, <blockquote.challenge>, <blockquote.challenge> ]
~~~
{: .output}


> ## Select the "Introduction" title by ID
> In a previous challenge, we were able to select the "Introduction" title because we knew it was
> the first `h1` element on the page. But what if we didn't know how many such elements were on the
> page. In other words, is there a different attribute that allows us to uniquely identify that title
> element?
>
> Using the path expressions introduced above, rewrite your XPath query to select
> the "Introduction" title without using the `[1]` index notation.
>
> Tips:
>
> * Look at the source of the page or use the "Inspect element" function of your browser to see what
>   other information would enable us to uniquely identify that element.
> * The syntax for selecting an element like `<div id="mytarget">` is `div[@id = 'mytarget']`.
>
>
> > ## Solution
> >
> > ~~~
> > $x("/html/body/div/h1[@id='introduction']")
> > ~~~
> > {: .source}
> >
> > should produce something similar to
> >
> > ~~~
> > <- Array [ <h1#introduction> ]
> > ~~~
> > {: .output}
> >
> {: .solution}
{: .challenge}




> ## Select this challenge box
> Using an XPath query in the JavaScript console of your browser, select the element that contains the text
> you are currently reading on this page.
>
> Tips:
>
> * In principle, `id` attributes in HTML are unique on a page. This means that if you know the `id`
>   of the element you are looking for, you should be able to construct an XPath that looks for this value
>   without having to worry about where in the node tree the target element is located.
> * The syntax for selecting an element like `<div id="mytarget">` is `div[@id = 'mytarget']`.
> * Remember that XPath queries are relative to a context node, and by default that node is the root node.
> * Use the `//` syntax to select for elements regardless of where they are in the tree.
> * The syntax to select the parent element relative to a context node is `..`
> * The `$x(...)` JavaScript syntax will always return an array of nodes, regardless of the number of
>   nodes returned by the query. Contrary to XPath, JavaScript uses _zero based indexing_, so the syntax to get
>   the first element of that array is therefore `$x(...)[0]`.
>
> Make sure you select this entire challenge box. If the result of your query displays only the title of
> this box, have a second look at the HTML structure of the document and try to figure out how to "expand"
> your selection to the entire challenge box.
>
> > ## Solution
> > Let's have a look at the HTML code of this page, around this challenge box (using the "View Source" option)
> > in our browser). The code looks something like this:
> >
> > ~~~
> > <!doctype html>
> > <html lang="en">
> >   <head>
> >     (...)
> >   </head>
> >   <body>
> > 	<div class="container">
> > 	(...)
> > 	  <blockquote class="challenge">
> > 	    <h2 id="select-this-challenge-box">Select this challenge box</h2>
> > 	    <p>Using an XPath query in the JavaScript console of your browser...</p>
> > 	    (...)
> > 	  </blockquote>
> > 	(...)
> > 	</div>
> >   </body>
> > </html>
> > ~~~
> > {: .output}
> >
> > We know that the `id` attribute should be unique, so we can use this to select the `h2` element inside
> > the challenge box:
> >
> > ~~~
> > $x("//h2[@id = 'select-this-challenge-box']/..")[0]
> > ~~~
> > {: .source}
> >
> > This should return something like
> >
> > ~~~
> > <- <blockquote class="challenge">
> > ~~~
> > {: .output}
> >
> > Let's walk through that syntax:
> >
> > |-----------------|:-------------|
> > | `$x("`| This function tells the browser we want it to execute an XPath query. |
> > | `//`| Look anywhere in the document... |
> > | `h2`| ... for an h2 element ... |
> > | `[@id = 'select-this-challenge-box']`| ... that has an `id` attribute set to `select-this-challenge-box`... |
> > | `//`| and select the parent node of that h2 element |
> > | `")"`| This is the end of the XPath query. |
> > | `[0]`| Select the first element of the resulting array (since `$x()` returns an array of nodes and we are only interested in the first one).|
> >
> > By hovering on the object returned by your XPath query in the console, your browser should helpfully highlight
> > that object in the document, enabling you to make sure you got the right one:
> >
> > ![Hovering over a resulting node in Firefox]({{ page.root }}/fig/firefox-hover.png)
> {: .solution}
{: .challenge}

# Advanced XPath syntax

FIXME: All the content below is from the original XPath lesson. Adapt content to use current example.

## Operators

Operators are used to compare nodes. There are mathematical operators, boolean operators. Operators can give you boolean (true/false values) as a result. Here are some useful ones:

| Operator   | Explanation |
|-----------------|:-------------|
|```=```|Equivalent comparison, can be used for numeric or text values|
|```!=```|Is not equivalent comparison|
|```>, >=```|Greater than, greater than or equal to|
|```<, <=```|Less than, less than or equal to|
|```or```|Boolean or|
|```and```|Boolean and|
|```not```|Boolean not|

### Examples

| Path Expression   | Expression Result |
|-----------------|:-------------|
|html/body/div/h3/@id='exercises-2'|Does exercise 2 exist?|
|html/body/div/h3/@id!='exercises-4'|Does exercise 4 not exist?|
|//h1/@id='references' or @id='introduction'|Is there an h1 references or introduction?|


## Predicates

Predicates are used to find a specific node or a node that contains a specific value.

Predicates are always embedded in square brackets, and are meant to provide additional filtering information to bring back nodes. You can filter on a node by using operators or functions.


### Examples

| Operator   | Explanation |
|-----------------|:-------------|
| ```[1]```  |Select the first node|
| ```[last()]```  |Select the last node|
| ```[last()-1]```  |Select the last but one node (also known as the second last node)|
|```[position()<3]```|Select the first two nodes, note the first position starts at 1, not = |
|```[@lang]```|	Select nodes that have attribute 'lang'|
|```[@lang='en']```|	Select all the nodes that have a "attribute" attribute with a value of "en"|
|```[price>15.00]```|	Select all nodes that have a price node with a value greater than 15.00|

### Examples

| Path Expression   | Expression Result |
|-----------------|:-------------|
|//h1[2]|Select 2nd h1|
|//h1[@id='references' or @id='introduction']|Select h1 references or introduction|

<!--
Note: '!=' != 'not', for instance in this snippet

```xml
<book id='bk101'></book>
<book></book>
<book type='paperback'></book>
<book id='bk102'></book>
<book id='bk103'></book>
```

The ```[@id != 'bk101']``` will bring back

```xml
<book id='bk102'></book>
<book id='bk103'></book>
```

While ```[not(@id='bk101')]``` will bring back

```xml
<book></book>
<book type='paperback'></book>
<book id='bk102'></book>
<book id='bk103'></book>
```

This is because != indicates the existence of an @id, whie the not() expression expresses to bring back everything except @id='bk101'
-->




## Wildcards

XPath wildcards can be used to select unknown XML nodes.

|Wildcard	|Description|
|-----------------|:-------------|
|```*```	|Matches any element node|
|```@*```|	Matches any attribute node|
|```node()```|	Matches any node of any kind|


### Examples

|Path Expression|	Result|//*[@id="examples-2"]
|-----------------|:-------------|
|```//*[@class='solution']```|Select all elements with class attribute 'solution'|


## In-text search

XPath can do in-text searching using functions and also supports regex with its matches() function. Note: in-text searching is case-sensitive!

|Path Expression|	Result|
|-----------------|:-------------|
|```//author[contains(.,"Matt")]```| Matches on all author nodes, in current node contains Matt (case-sensitive)|
|```//author[starts-with(.,"G")]```|Matches on all author nodes, in current node starts with G (case-sensitive)|
|```//author[ends-with(.,"w")]```|Matches on all author nodes, in current node ends with w (case-sensitive)|
|```//author[matches(.,"Matt.*")]```| regular expressions match 2.0 |

<!--
### Exercises

| Expression   | Result |
|-----------------|:-------------|
|||
-->

## Complete syntax: XPath Axes

XPath Axes fuller syntax of how to use XPath. Provides all of the different ways to specify the path by describing more fully the relationships between nodes and their connections. The XPath specification describes 13 different axes:

* self ‐‐ the context node itself
* child ‐‐ the children of the context node
* descendant ‐‐ all descendants (children+)
* parent ‐‐ the parent (empty if at the root)
* ancestor ‐‐ all ancestors from the parent to the root
* descendant‐or‐self ‐‐ the union of descendant and self • ancestor‐or‐self ‐‐ the union of ancestor and self
* following‐sibling ‐‐ siblings to the right
* preceding‐sibling ‐‐ siblings to the left
* following ‐‐ all following nodes in the document, excluding descendants
* preceding ‐‐ all preceding nodes in the document, excluding ancestors • attribute ‐‐ the attributes of the context node

[![XPath Axes Image Credit: SAMS Teach Yourself XSLT in 21 Days](https://kimpham54.github.io/library-webscraping/fig/xpath-axes.jpg)]({{ page.root }}/fig/xpath-axes.jpg)


|Path Expression|	Result|
|-----------------|:-------------|
|```/html/body/div/h1[@id='introduction']/following-sibling::h1```|Select all h1 following siblings of the h1 introduction|
|```/html/body/div/h1[@id='introduction']/following-sibling::*```|Select all h1 following siblings|
|```//attribute::id```|Select all id attribute nodes|

Oftentimes, the elements we are looking for on a page have no ID attribute or
other uniquely identifying features, so the next best thing is to aim for
neighboring elements that we can identify more easily and then use node
relationships to get from those easy to identify elements to the target elements.

For example, the node tree image above has no uniquely identifying feature like an ID attribute.
However, it is just below the section header "Navigating through the HTML node tree using XPath".
Looking at the source code of the page, we see that that header is a `h2` element with the id
`navigating-through-the-html-node-tree-using-xpath`.

~~~
$x("//h2[@id='navigating-through-the-html-node-tree-using-xpath']/following-sibling::p[2]/img")
~~~
{: .source}


# Additions

FIXME: add more XPath functions such as concat() and normalize-space().
FIXME: mention [XPath Checker for Firefox](https://addons.mozilla.org/en-US/firefox/addon/xpath-checker/)
FIXME: Firefox sometime cleans up the HTML of a page before displaying it, meaning that the DOM tree
we can access through the console might not reflect the actual source code. `<tbody>` elements are
typically not reliable.
The [Scrapy documentation](https://doc.scrapy.org/en/latest/topics/firefox.html#caveats-with-inspecting-the-live-browser-dom)
has more on the topic. 

# References

* [W3Schools: JavaScript HTML DOM Navigation](http://www.w3schools.com/js/js_htmldom_navigation.asp)
* [XPath Cheatsheet](/library-webscraping/extras/xpath-cheatsheet.md.pdf)
