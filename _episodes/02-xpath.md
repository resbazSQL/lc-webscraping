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

# Introduction

Consider the exercise we did in the prior section. It required us to open the page and interact with the browser. Either by right-clicking on what we wanted to find or using ctrl-f to look for a text string. This sort of interaction doesn't scale, since it requires one of us at the controls to search and extract data. The rest of this session will be looking at ways to tell the computer what to find, so that we don't have to interact with the page directly. 

While there are many ways to interact with a webpage, XPath is *designed* to work with XML based languages and is therefore a universal option, if somewhat difficult to write. All of the other mechanisms, however, just (usually) hide xpath queries behind various interfaces -- so it's important to have the theoretical model in mind before moving on to easier interactions.

Before we delve into web scraping proper, we will first spend some time introducing
some of the techniques that are required to indicate, *to the computer*, exactly what should be
extracted from the web pages we aim to scrape.

The material in this section was adapted from the [XPath and XQuery Tutorial](https://github.com/code4libtoronto/2016-07-28-librarycarpentrylessons/blob/master/xpath-xquery/lesson.md)
written by [Kim Pham](https://github.com/kimpham54) ([@tolloid](https://twitter.com/tolloid))
for the July 2016 [Library Carpentry workshop](https://code4libtoronto.github.io/2016-07-28-librarycarpentry/) in Toronto.



## XPath

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
> HTML is just a particular dialect of XML. 
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

