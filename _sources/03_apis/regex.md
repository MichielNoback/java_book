# Regular Expressions

Often, when your data is well-structured, a `String.split()` with a simple separator is all you need to process your data,maybe with some conversion methods for creating ints and doubles and such from strings. However, there are cases when you need more: finding restriction enzyme sites, parsing zip codes in addresses for instance.

This part only introduces the Java regex API
It is NOT a tutorial on regex!
(PS, I borrowed some examples from http://www.vogella.com)

Regular expressions are used to search for patterns in text
For instance, if you want to find occurrences of Dutch zip codes, you could use this: `[a-zA-Z]{4} ?[0-9]{2}`

For Java regex, these classes are related to regular expression matching (and replacement):  

- `java.lang.String` has several useful methods working with regexes
- `java.util.regex.Pattern` works together with `java.util.regex.Matcher` for advanced regex operations


## Regex syntax

A regular expression is used to describe a pattern that is not literal - different distinct character strings could match the pattern. 

### Backslashes

Backslashes have meanings both in String literals and in regex. They denote special characters, but also negate the special meaning
Thus, to match a literal backslash, your regex will be `\\\\` !! 

### Character classes

You can specify groups of characters that are all equally valid to match a position using character classes. They can be specified in many ways.

| Character class | Description                                                                                                        |
|-----------------|--------------------------------------------------------------------------------------------------------------------|
| .               | Any character                                                                                                      |
| []              | Set definition, eg [a-z] matches all lowercase characters;  [aAbB1234] matches an a, A, b, B or numbers 1, 2 ,3, 4 |
| [&#94;]             | Negated set definition, eg [&#94;a-z] matches anything BUT lowercase characters                                        |
| X&#124;Z             | Matches X or Z (either one will suffice)                                                                           |
| &#94;regex          | Anchors regex at beginning of line                                                                                 |
| regex$          | Anchors regex at end of line                                                                                       |
| \d \D           | Any digit, [0-9];  any non-digit, [&#94;0-9]                                                                           |
| \s \S           | Any whitespace character;  any non-whitespace character                                                            |
| \w \W           | A word character, short for [a-zA-Z_0-9];  same but negated                                                        |
| \b              | Word boundary                                                                                                      |

### Regex quantifiers

Quantifiers let you modify how often a group or character is allowed.

| Quantifier | Description                                                                                              |
|------------|----------------------------------------------------------------------------------------------------------|
| {x}        | Occurs exactly x times                                                                                   |
| {x, }      | Occurs at least x times                                                                                  |
| {, x}      | Occurs at the most x times                                                                               |
| *          | Occurs zero or more times; same as {0, }                                                                 |
| +          | Occurs one or more times; same as {1, }                                                                  |
| ?          | Occurs zero or one time; same as {0, 1}                                                                  |
| *?         | Non-greedy: "?" after a quantifier makes it a reluctant quantifier, it tries to find the smallest match |


### Grouping and backreferencing

Using parentheses, you can group parts of your regex. 
They can be used to
- Retrieve or substitute parts of a regex
- Apply quantifiers to groups
- use back referencing

Via the `$` you can refer to a group. `$1` is the first group, `$2` the second, etc (see examples).

Similarly, back referencing is used to repeat pattern matches; this will replace all repeating character patterns:

```java
"ABCDEEEFGGHIJJJKL".replaceAll("(\\w)\\1+", "_rep_")
```

resulting in `ABCD_rep_F_rep_HI_rep_KL`.


## Methods of class String

Many of the common tasks can be performed using the String class only. 

| Method                             | Description                                                                                                       |
|------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| s.matches(regex)                   | Tells whether or not this string matches the given regular expression.                                            |
| s.split(regex)                     | Splits this string around matches of the given regular expression                                                 |
| s.split(regex, limit)              | Idem, but the `limit` parameter controls the number of times the pattern is applied                                 |
| s.replaceAll(regex, replacement)   | Replaces each substring of this string that matches the given regular expression with the given replacement       |
| s.replaceFirst(regex, replacement) | Replaces the first substring of this string that matches the given regular expression with the given replacement. |

Here are some examples.

```java
String input = "Dogs rule this doggin' world";
//replace() works with literal string!
System.out.println(input.replace("[Dd]og", "Cat"));
System.out.println(input.replace("Dog", "Cat"));
//replaceAll() works with regex!
System.out.println(input.replaceAll("[Dd]og", "Cat"));
System.out.println(Arrays.toString(input.split("[Dd]")));
//matches() looks at whole target string.
System.out.println(input.matches("[Dd]ogs"));
System.out.println(input.matches("^[Dd]ogs.+"));
```

outputs 


<pre class="console_out">
Dogs rule this doggin' world
Cats rule this doggin' world
Cats rule this Catgin' world
[, ogs rule this , oggin' worl]
false
true
</pre>

Some more advanced examples:

```java
String INPUT = "This is the <title>example</title> " 
	+ "string which , I'm going to use for pattern matching.";

// Split on whitespace stretches
String[] splitString = (INPUT.split("\\s+"));

// Removes whitespace between a word character and . or ,
String pattern = "(\\w)(\\s+)([\\.,])";
System.out.println(INPUT.replaceAll(pattern, "$1$3")); 

// Extract the text between the two title elements of
pattern = "(?i)(<title.*?>)(.+?)(</title>)";
String updated = INPUT.replaceAll(pattern, "$2"); 

// prints true if the string contains a number less then 300
System.out.println(s.matches("[^0-9]*[12]?[0-9]{1,2}[^0-9]*"));
```

Note that adjusting the regex mode with `(?i)<regex>` makes the regex case insensitive. 

## Pattern & Matcher

For more complex tasks, use class `Pattern` to specify it and class `Matcher` to find, replace or extract it.

Here is a typical use case:

```java
        //with case insensitive matching
        Pattern hinc2 = Pattern.compile("(?i)(GT([CT][AG])AC)");
        Matcher matcher = hinc2.matcher("GTCAACtgttgaccc");

        while (matcher.find()) {
            System.out.println("matcher.group() = " + matcher.group());
            //whole pattern - same as group()
            System.out.println("matcher.group(0) = " + matcher.group(0));
            System.out.println("matcher.group(2) = " + matcher.group(2));
            System.out.println("matcher.start() = " + matcher.start());
        }

        //with chained method calls - replace with group capture
        final String replaced = hinc2.matcher("GTCAACtgttgaccc").replaceAll("[[$1]]");
        System.out.println("replaced = " + replaced);
```

outputs

<pre class="console_out">
matcher.group() = GTCAAC
matcher.group(0) = GTCAAC
matcher.group(2) = CA
matcher.start() = 0
matcher.group() = gttgac
matcher.group(0) = gttgac
matcher.group(2) = tg
matcher.start() = 7
replaced = [[GTCAAC]]t[[gttgac]]cc
</pre>

Note: **when a `Pattern` object is used more than once, a compiled version should be cashed!** Usually this is done in an instance or class variable.

There is much more to regex of course; GIYF for more detailed use cases.
