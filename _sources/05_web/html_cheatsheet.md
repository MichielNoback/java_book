# HTML5 tags

## Introduction

Below is an alphabetic listing of the most important (most-used) html5 tags that may be relevant on the test of the module webbased information systems 1. The information presented in this page is mostly copied from the site [w3schools](https://www.w3schools.com).

## The tags

### Links: `<a>`

The `<a>` tag defines a hyperlink, which is used to link from one page to another.
Most important attribute is `href` which specifies the URL the link goes to.

Example

```html
<a href="https://www.w3schools.com">Visit W3Schools.com!</a>
```
<a href="https://www.w3schools.com">Visit W3Schools.com!</a>

### Page body: `<body>`

The `<body>` tag defines the document's body.
The `<body>` element contains all the contents of an HTML document, such as text, hyperlinks, images, tables, lists, etc.

```html
<html>
    <head>
    <title>Title of the document</title>
    </head>

    <body>
    The content of the document......
    </body>

</html>
```

### Line breaks: `<br>`

The `<br>` tag inserts a single line break.

The `<br>` tag is an empty tag which means that it has no end tag.

In HTML, the `<br>` tag has no end tag.
In XHTML, the <br> tag must be properly closed, like this: `<br />`.

```html
This text contains<br>a line break.
```

This text contains<br>a line break.

### A clickable button: `<button>`

Inside a `<button>` element you can put content, like text or images. 
This is the difference between this element and buttons created with the `<input>` element.

```html
<button type="button">Click Me!</button>
```
<button type="button">Click Me!</button>


The most important attributes are
- `value`, which specifies the initial value (text).
- `type`, the type that can be either `button`, `reset` or `submit`

### Predefined text types

The `<code>` tag is a phrase tag. It defines a piece of computer code.Other simalar types of tags are included in the fragment below.

```html
<em>Emphasized text</em><br>
<strong>Strong text</strong><br>
<code>A piece of computer code</code><br>
<samp>Sample output from a computer program</samp><br>
<kbd>Keyboard input</kbd><br>
<var>Variable</var>
```

<em>Emphasized text</em><br>
<strong>Strong text</strong><br>
<code>A piece of computer code</code><br>
<samp>Sample output from a computer program</samp><br>
<kbd>Keyboard input</kbd><br>
<var>Variable</var>

Here are all "phrase" tags:

- `<em>`	Renders as emphasized text
- `<strong>`	Defines important text
- `<code>`	Defines a piece of computer code
- `<samp>`	Defines sample output from a computer program
- `<kbd>`	Defines keyboard input
- `<var>`	Defines a variable

### Page section: `<div>`

The `<div>` tag defines a division or a section in an HTML document.

The `<div>` element is often used as a container for other HTML elements to style them with CSS or to perform certain tasks with JavaScript.

```html
<div style="background-color:lightblue">
  <h4>This is a heading</h4>
  <p>This is a paragraph.</p>
</div>
```

<div style="background-color:lightblue">
  <h4>This is a heading</h4>
  <p>This is a paragraph.</p>
</div>

### A description list: `<dl>`, `<dd>` &amp; `<dt>`

A description list, with terms and descriptions.
The `<dl>` tag defines a description list.

The `<dl>` tag is used in conjunction with `<dt>` (defines terms/names) and `<dd>` (describes each term/name).

```html
<dl>
  <dt>Coffee</dt>
  <dd>Black hot drink</dd>
  <dt>Milk</dt>
  <dd>White cold drink</dd>
</dl>
```

<dl>
  <dt>Coffee</dt>
  <dd>Black hot drink</dd>
  <dt>Milk</dt>
  <dd>White cold drink</dd>
</dl>

### Forms and its elements

The `<form>` tag is used to create an HTML form for user input.
The `<form>` element can contain one or more of the following form elements:

- `<input>`
- `<textarea>`
- `<button>`
- `<select>`
- `<option>`
- `<label>`

(there are more but these are not relevant for this course).
The most important attributes are its `action` (the URL to submit to), its `method` (GET or POST) and its `enctype` when submitting for file uploads.
Here is a typical form demonstrating these elements. Note the different attributes for the `input` element.

```html
    <form action = "#", method = "post">
        <label for="username-field">Username</label>:<br>
        <input id="username-field" type="text" name="username" minlength="5" maxlength="20"><br>
        <label for="password-field">Username</label>:<br>
        <input id="password-field" type="password" name="password" minlength="5" maxlength="20"><br>
        <label for="email-field">Email</label>:<br>
        <input id="email-field" type="email" name="email"><br>
        <br>
        Favourite animals (more than one can be selected)<br>
        <select name="favorite-animal" size="3" multiple>
            <option name="cheetah">Cheetah</option>
            <option name="lion">Lion</option>
            <option name="platypus">Platypus</option>
            <option name="goat">Goat</option>
            <option name="coyote">Coyote</option>
            <option name="skunk" selected>Skunk</option>
        </select>
        <br><br>
        <textarea name="message" rows="10" cols="30">My favourite animal is the &lt;fill in&gt; because (FINISH).
        </textarea>
        <br>
        Gender:
        <br>
        <input id="gendermale" type="radio" name="gender" value="male">
        <label for="gendermale">Male</label>
        <input id="genderfemale" type="radio" name="gender" value="female">
        <label for="genderfemale">Female</label>
        <input id="genderother" type="radio" name="gender" value="other">
        <label for="genderother">Other</label>
        <br>
        <input type="submit" value="That's it!">
    </form>
<hr>
```

<form action = "#", method = "post">
    <label for="username-field">Username</label>:<br>
    <input id="username-field" type="text" name="username" minlength="5" maxlength="20"><br>
    <label for="password-field">Username</label>:<br>
    <input id="password-field" type="password" name="password" minlength="5" maxlength="20"><br>
    <label for="email-field">Email</label>:<br>
    <input id="email-field" type="email" name="email"><br>
    <br>
    Favourite animals (more than one can be selected)<br>
    <select name="favorite-animal" size="3" multiple>
        <option name="cheetah">Cheetah</option>
        <option name="lion">Lion</option>
        <option name="platypus">Platypus</option>
        <option name="goat">Goat</option>
        <option name="coyote">Coyote</option>
        <option name="skunk" selected>Skunk</option>
    </select>
    <br><br>
    <textarea name="message" rows="10" cols="30">My favourite animal is the &lt;fill in&gt; because (FINISH).
    </textarea>
    <br>
    Gender:
    <br>
    <input id="gendermale" type="radio" name="gender" value="male">
    <label for="gendermale">Male</label>
    <input id="genderfemale" type="radio" name="gender" value="female">
    <label for="genderfemale">Female</label>
    <input id="genderother" type="radio" name="gender" value="other">
    <label for="genderother">Other</label>
    <br>
    <input type="submit" value="That's it!">
</form>

<hr>

### Headers: `<h1>` - `<h6>`

The default appearance of headers may vary between browsers.


```html
<h1>This is heading 1</h1>
<h2>This is heading 2</h2>
<h3>This is heading 3</h3>
<h4>This is heading 4</h4>
<h5>This is heading 5</h5>
<h6>This is heading 6</h6>
```


### The non-rendered part of the DOM: `<head>`

The `head` element is a container for all the head elements.
The `head` element can include a title for the document, scripts, styles, meta information, and more.
The following elements can go inside the `head` element:  

- `title` (this element is required in an HTML document)
- `style`
- `base`
- `link`
- `meta`
- `script`
- `noscript`  


### Separator for sections: `<hr>`

```html
<h4>This is before</h4>
<hr>
<h4>This is after</h4>
```

<h4>This is before</h4>
<hr>
<h4>This is after</h4>

### The top level container: `<html>`

The `html` tag tells the browser that this is an HTML document.
The `html` tag represents the root of an HTML document.
The `html` tag is the container for all other HTML elements (except for the `!DOCTYPE` tag).

### Images: `<img>`

The `<img>` tag defines an image in an HTML page.
The `<img>` tag has two required attributes: src and alt.
Images are not technically inserted into an HTML page, images are linked to HTML pages. The `<img>` tag creates a holding space for the referenced image.
Tip: To link an image to another document, simply nest the `<img>` tag inside `<a>` tags.


### Describe metadata: `<meta>`

Metadata is data (information) about data.
The `<meta>` tag provides metadata about the HTML document. Metadata will not be displayed on the page, but will be machine parsable.  
Meta elements are typically used to specify page description, keywords, author of the document, last modified, and other metadata.  
The metadata can be used by browsers (how to display content or reload page), search engines (keywords), or other web services.  
HTML5 introduced a method to let web designers take control over the viewport (the user's visible area of a web page), through the `<meta>` tag.

```html
<head>
  <meta charset="UTF-8">
  <meta name="description" content="Free Web tutorials">
  <meta name="keywords" content="HTML,CSS,XML,JavaScript">
  <meta name="author" content="John Doe">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
```

### Ordered and unordered lists: `<ol>`, `<ul>`, `<li>`

Ordered lists have numbering, unordered lists are not (usually bulleted lists).

The main attributes of an ordered list are:  
- `reversed`
- `start` the start number
- `type` the number type. Possible values are `1` (the default), `A`, `a`, `I`, `i`. 

```html
<ol start="10", type="i">
  <li>Coffee</li>
  <li>Tea</li>
  <li>Milk</li>
</ol>
```
<ol start="10", type="i">
  <li>Coffee</li>
  <li>Tea</li>
  <li>Milk</li>
</ol>

The unordered list can only configure its symbol, but this is not generally supported in html5.

```html
<ul type="square">
  <li>Coffee</li>
  <li>Tea</li>
  <li>Milk</li>
</ul>
```

<ul type="square">
  <li>Coffee</li>
  <li>Tea</li>
  <li>Milk</li>
</ul>

### Paragraphs: `<p>`

The `<p>` tag defines a paragraph.  
Browsers automatically add some space (margin) before and after each `<p>` element. The margins can be modified with CSS (with the margin properties).

```html
<p style="border-style: dotted; text-align: center">This is some text in a paragraph.</p>
```
<p style="border-style: dotted; text-align: center">This is some text in a paragraph.</p>

### Scripts with `<script>`

Usually you place scripts outside the html, but there are good uses for 
placing the code within script tags.

```html
<div id="demo">Hello World</div>
<script>
document.getElementById("demo").innerHTML = "Hello JavaScript!";
</script>
```

### Tables: `<table>`

The `<table>` tag defines an HTML table.
An HTML table consists of the `<table>` element and one or more `<tr>`, `<th>`, and `<td>` elements.
The `<tr>` element defines a table row, the `<th>` element defines a table header, and the `<td>` element defines a table cell.
A more complex HTML table may also include `<caption>`, `<col>`, `<colgroup>`, `<thead>`, `<tfoot>`, and `<tbody>` elements.
Here is a table with the tags it can contain:

```html
<table>
    <thead>
    <tr>
        <th>Nucleotide</th>
        <th>Name</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>A</td>
        <td>Adenine</td>
    </tr>
    <tr>
        <td>C</td>
        <td>Cytosine</td>
    </tr>
    <tr>
        <td>G</td>
        <td>Guanine</td>
    </tr>
    <tr>
        <td>T</td>
        <td>Thymine</td>
    </tr>
    </tbody>
</table>
```

<table>
    <thead>
    <tr>
        <th>Nucleotide</th>
        <th>Name</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>A</td>
        <td>Adenine</td>
    </tr>
    <tr>
        <td>C</td>
        <td>Cytosine</td>
    </tr>
    <tr>
        <td>G</td>
        <td>Guanine</td>
    </tr>
    <tr>
        <td>T</td>
        <td>Thymine</td>
    </tr>
    </tbody>
</table>


