# Thymeleaf expressions

## Introduction

This post is one of a series on Thymeleaf. It deals with some of the essential templating techniques that Thymeleaf offers:

- using texts
- implicit objects (predefined variables)
- variables (of the context/model) 
- urls/links
- attributes available for templating

For a complete overview you should refer to [the official docs](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html).

## Expression types

### Resource expressions: `#{}`

Using the `#{message.in.resource.bundle}` expression will make the template engine look in your resource bundle(s) for internationalized messages with the given key. In its most basic form it works as follows.
This expression: 

```html
<h3 th:text="#{page.title}">_offline_</h3>
```

combined with this property in a resource bundle

```
page.title=get your phrase-of-the-day 
```

will evaluate to 

```html
<h3>get your phrase-of-the-day</h3>
```

#### Unescaped text

The Thymeleaf attribute `th:text` ‘escapes’ the text in the message. If you want this message: 

```
favourite=Thymeleaf is my <b>favourite</b> templating engine!
```

to display correctly, you should use `th:utext`:

```html
<h3 th:utext="#{page.title}">_offline_</h3>
```

So you’ll get this

```html
<p>Thymeleaf is my <b>favourite</b> templating engine!</p>
```

Instead of this 

```html
<p>Thymeleaf is my &lt;b&gt;favourite&lt;/b&gt; templating engine!</p>
```

#### Messages with variables

To insert variables into your messages, give the message identifier an argument, just like calling a function. Then, in the message "body", catch it with {0}.

Given this message

```properties
animal=The {0} is the scariest animal there is!
```

and this Thymeleaf

```html
<p th:text="#{animal('black widow')}"></p>
<p th:text="#{animal('rattlesnake')}"></p>
```

will process into

```html
<p>The black widow is the scariest animal there is!</p>
<p>The rattlesnake is the scariest animal there is!</p>
```

Of course, you should use Use {1}, {2} ... for multiple variables.

### Variable expressions: `${}`

Variables expressions is completely different from messages with variables - the previous section!

Variable expressions query the `WebContext`, or other implicit objects to resolve these expressions. Usually, they will refer to objects you have attached to the `WebContext` object in the servlet request handling process.

For example, this servlet code: 

```java
WebContext ctx = new WebContext(
        request,
        response,
        request.getServletContext(),
        request.getLocale());
ctx.setVariable("warning",
    "you must be over 18 to drive a car");
templateEngine.process("thymeleaf_demo", ctx, response.getWriter());
```

With this Thymeleaf expression in `thymeleaf_demo`

```html
<p th:text="${warning}">_offline_</p>
```

will evaluate to 

```html
<p> you must be over 18 to drive a car </p>
```

Thus, the `warning` attribute that was set on the `WebContext` object has been retrieved in the Thymeleaf template using the `${warning}` expression.

#### Using Java bean properties in expressions

A Java bean is a special kind of class, coded according to a set of rules. That’s why you can use them so conveniently in Thymeleaf, without actually writing Java code.

These are the JavaBean design rules:  

- They provide a no-arg constructor ( e.g. `public Person(){}` )
- They provide `public` standard-named getters and setters for class attributes; e.g. for `String` property `name` you must provide the methods `public String getName(){return "Fred"}` and `public void setName(String name){}`.

A very important thing to realize is the class does not necessary have to have the actual property (private String name); only the getters and setters are required!

Another thing to note is you are not required to provide a setter, as long as you don't use it in your Thymeleaf templates.

For example, here is a bean publishing the `fullName` property without actually having such a String variable internally:

```java
public class Person {
    private String firstName;
    private String lastName;

    public Person() {
    }

    public Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public String getFullName() {
        return firstName + " " + lastName;
    }
}
```

When set on the context like this:

```java
ctx.setVariable("person", new Person("Judy", "Steinberg"));
```

and this template fragment

```html
<span th:text="${'The person currently logged in is ' + person.fullName}">_offline_</span>
```

will give the processed html

```html
<span>The person currently logged in is Judy Steinberg</span>
```

#### Flexibility of `${}`

The `${}` expression can be used in many ways; here are only a few:
- Chaining of calls: **`${object.property.property}`**
- List access: **`${list[1]} or ${list[${selected_index}]}`**
- Map access: **`${map['foo']}`** or **`${map[${selected_item}].property}`**
- Regular method calls: **`${person.createCompleteNameWithSeparator('-')}`**
- Combinations of expression types

    ```html
    <h4 th:text="#{'phrase.' + ${phrase_type} + '.' + ${phrase_num}}">_phrase_</h4>
    ```

### Built-in-variable expressions: `${#}`

Thymeleaf provides quite a few predefined variables that can be accessed using `${#variableName}` syntax. Here are some:

**variable**|**purpose**
-----|-----
`#locale`|the context locale.
`#httpServletRequest`|(only in Web Contexts) the `HttpServletRequest` object. Access via `${param.key}` in Spring
`#dates`|utility methods for `java.util.Date` objects: formatting, component extraction, etc.
`#numbers`|utility methods for formatting numeric objects.
`#strings`|(or `#objects`, `#bools`, `#arrays`, `#lists`, `#sets`: utility methods for these objects

#### Locales

The '#locale' helper can do things like this:

Get the Language set on the clients' browser: `${#locale.getLanguage()}` will give `nl` on my browser.  
Get the Country set on the clients' browser; it will be empty when not set: `${#locale.getCountry()}` will give `<nothing>` on my browser.   
Get the display name of your locale set on your browser: `${#locale.getDisplayName()}` : `Dutch`

#### Dates and Calendars

The '#dates' and '#calendars' variables help with everything related to dates.  

Here are some examples, with `my_date`, a `java.util.Date` object, set on the web context model:

**Expression**|**Description**|**Result**
-----|-----|-----
`${my_date}`|the date|Mon Jun 03 15:52:03 CEST 2019
`${#dates.format(my_date)}`|simple date formatting using default locale|3 juni 2019 15:52:03 CEST
`${#dates.format(my_date, 'yyyy/mm/dd HH:mm')}`|advanced date formatting using format string|2019/52/03 15:52
`${#dates.formatISO(my_date)}`|ISO format|2019-06-03T15:52:03.981+02:00

#### Numbers

The '#numbers' implicit object helps with numbers. 
Given 'my_date' set on the web context model: `ctx.setVariable("my_number", 31415.9265359);`, here is simple usage scenarios.

`${my_number}` gives "31415.9265359", the exact representation of the context variable.  

`${#numbers.formatDecimal(my_number, 0, 'COMMA', 2, 'POINT')} ` gives "31,415.93".  
The argument are as follows:   
`#numbers.formatDecimal(<target>, <mimimum integer digits>, '<thousandspoint type>', <decimal digits>, <decimalpoint type>)`   

Similarly, this expression `${#numbers.formatInteger(my_number, 10, 'POINT')}` will produce "0.000.031.416".

#### The session object

The session object is simply available as `session`.
The expression `${session.user.email}` will display the 'email' property of the User object registered as attribute on the 'session' object: `session.setAttribute("user", new User("Henk", "henk@example.com", Role.USER));` will give
"henk@example.com" 

The details of sessions are the topic of a separate post, however.

### Locally scoped variables with `*{}`

You can access the properties of a locally scoped object through the `*{}` syntax. This obviates the need for retyping the variable name over and over again.  

Given the User object of the session example, you can list its properties as follows:

```html
<ul th:object="${session.logged_in_user}">
<li>Name: <span th:text="*{name}">_name_</span></li>
<li>Email: <span th:text="*{email}">_email_</span></li>
<li>Role: <span th:text="*{role}">_role_</span></li>
</ul>
```

Which will produce
```html
<ul>
    <li>Name: <span>Henk</span></li>
    <li>Email: <span>henk@example.com</span></li>
    <li>Role: <span>USER</span></li>
</ul>
```

### Creating links: `@{}`

Using the `@{}` syntax, you can create links with a path that is relative to the **_deployment context_**.
```html
<a th:href="@{/give.phrase(phrase_category=bullshit)}" href="#">get a bullshit phrase</a>
```

will give the following link: http://localhost:8080/give.phrase?phrase_category=bullshit.

As you can see, you can add a request parameter as key=value pair. For multiple key=value pairs, separate them by comma's: `@{/give.phrase(phrase_category=bullshit,action=nothing)}` gives http://localhost:8080/give.phrase?phrase_category=bullshit&action=nothing

### html tag attributes you can access

You have seen Thymeleaf targeting the "text" attribute a lot now, using `th:text`. But there are many many more. Have a look at the Thymeleaf docs for a complete listing.
Here are the main ones:

**attribute**|**target**
-----|-----
th:action|the action of a form
th:class|the style class of an element
th:href|the url a link will point to
th:id|the ID of an element
th:name|the name of an element
th:rel|url for style sheets
th:src|the source of an element (e.g. image)
th:title|the title
th:value|the value of a (form) element, e.g.
th:attr|a generic means to define any attribute, e.g. th:attr="dat=#{subscribe.submit}"

An example. This Servlet code

```java
ctx.setVariable("name_of_bird_selector", "fav_birds");
ctx.setVariable("bird_groups", List.of("raptors", "songbirds", "waders", "wildfowl"));
```

and this Thymeleaf template fragment

```html
<select th:name="${name_of_bird_selector}">
    <option th:each="bird_cat:${bird_groups}"
            th:value="${bird_cat}"
            th:text="'The ' + ${bird_cat}"
            th:id="${bird_cat + '_option'}"
            value="_bird_cat_">_bird_cat_</option>
</select>
```

produce this html

```html
<select name="fav_birds">
    <option id="raptors_option" value="raptors">The raptors</option>
    <option id="songbirds_option" value="songbirds">The songbirds</option>
    <option id="waders_option" value="waders">The waders</option>
    <option id="wildfowl_option" value="wildfowl">The wildfowl</option>
</select>
```

### Inline expressions

Sometimes, it is not desirable to have your variables expressed within tag attributes. Instead, you want raw (html) text. This is especially the case with Javascript, when "injecting" variable values. Here is its main usage scenario:

```html
<button id="say_hello">Say hello!</button>

<script th:inline="javascript">
    var user = [[${session.logged_in_user.name}]];
    document.getElementById("say_hello").onclick = function() {
        alert("Hi, " + user);
    }
</script>
```

Although this is a typical use case, you can also use `[[${expr}]]` whenever you want some value inserted in a textual context, without using `<span th:text="${...}">`.
