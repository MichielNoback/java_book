# Thymeleaf flow control and fragments

## Introduction

This post is the second on Thymeleaf. It deals with these templating techniques that Thymeleaf offers:

- conditionals 
- loops
- using fragments as building blocks

For a complete overview you should refer to [the official docs](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html "Thymeleaf documentation").

These examples all make use of the `Movie` class:

```java
public class Movie {
    private String title;
    private int year;
    private double rating = -1;
    private Set<String> mainActors = new HashSet<>();
    private static List<Movie> movies;

    public Movie() {
    }

    public Movie(String title, int year) {
        this.title = title;
        this.year = year;
    }

    public Movie(String title, int year, double rating) {
        this.title = title;
        this.year = year;
        this.rating = rating;
    }

    public Movie(String title, int year, double rating, List<String> mainActors) {
        this.title = title;
        this.year = year;
        this.rating = rating;
        this.mainActors.addAll(mainActors);
    }

    public String getTitle() {
        return title;
    }

    public int getYear() {
        return year;
    }

    public double getRating() {
        return rating;
    }

    public List<String> getMainActors() {
        List<String> actors = new ArrayList<>();
        actors.addAll(mainActors);
        actors.sort(String::compareTo);
        return actors;
    }

    public void setRating(double rating) {
        this.rating = rating;
    }

    public void addActor(String actor) {
        mainActors.add(actor);
    }

    public static List<Movie> getAllMovies() {
        return Movie.movies;
    }

    //create a list from IMDB top 20
    static {
        List<Movie> movies = new ArrayList<>();
        Movie movie;
        movie = new Movie("The Shawshank Redemption", 1994, 9.2, List.of("Tim Robbins", "Morgan Freeman"));
        movies.add(movie);
        movie = new Movie("The Dark Knight", 2008, 9.0, List.of("Christian Bale,", "Heath Ledger"));
        movies.add(movie);
        movie = new Movie("Pulp Fiction", 1994, 8.9, List.of("John Travolta", "Uma Thurman", "Samuel L. Jackson"));
        movies.add(movie);
        movie = new Movie("Fight Club ", 1999, 8.8, List.of("Brad Pitt", "Edward Norton"));
        movies.add(movie);
        movie = new Movie("Forrest Gump", 1994, 8.7, List.of("Tom Hanks", "Robin Wright", "Gary Sinise"));
        movies.add(movie);
        movie = new Movie("Inception", 2010, 8.7, List.of("Leonardo DiCaprio", "Joseph Gordon-Levitt"));
        movies.add(movie);
        movie = new Movie("One Flew Over the Cuckoo's Nest", 1975, 8.7, List.of("Jack Nicholson", "Louise Fletcher", "Will Sampson"));
        movies.add(movie);
        movie = new Movie("The Usual Suspects", 1995, 8.5, List.of("Kevin Spacey", "Gabriel Byrne", "Chazz Palminteri"));
        movies.add(movie);
        Movie.movies = movies;
    }
}
```

The list of movies is attached to the WebContxt object using

```java
ctx.setVariable("movies", Movie.getAllMovies());
```

so for all examples the `$movies` variable is available.


## Iteration

Iteration with Thymeleaf is as simple as in Java code; simply use `th:each`.
This is an example with the movies list: 

```html
<table>
    <thead>
    <tr>
        <th>Title</th>
        <th>Year of release</th>
        <th>IMDB rating</th>
    </tr>
    </thead>
    <tbody>
    <tr th:each="movie:${movies}">
        <td th:text="${movie.title}">_title_</td>
        <td th:text="${movie.year}">_year_</td>
        <td th:text="${movie.rating}">_rating_</td>
    </tr>
    </tbody>
</table>
```

Alternatively, you can use the **_local scope_** syntax:

```html
    <tr th:each="movie:${movies}" th:object="${movie}">
        <td th:text="*{title}">_title_</td>
        <td th:text="*{year}">_year_</td>
        <td th:text="*{rating}">_rating_</td>
    </tr>
```

both of which result in this table:

<table>
    <thead>
    <tr>
        <th>Title</th>
        <th>Year of release</th>
        <th>IMDB rating</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>The Shawshank Redemption</td>
        <td>1994</td>
        <td>9.2</td>
    </tr>
    <tr>
        <td>The Dark Knight</td>
        <td>2008</td>
        <td>9.0</td>
    </tr>
    <tr>
        <td>Pulp Fiction</td>
        <td>1994</td>
        <td>8.9</td>
    </tr>
    <tr>
        <td>Fight Club </td>
        <td>1999</td>
        <td>8.8</td>
    </tr>
    <tr>
        <td>Forrest Gump</td>
        <td>1994</td>
        <td>8.7</td>
    </tr>
    <tr>
        <td>Inception</td>
        <td>2010</td>
        <td>8.7</td>
    </tr>
    <tr>
        <td>One Flew Over the Cuckoo's Nest</td>
        <td>1975</td>
        <td>8.7</td>
    </tr>
    <tr>
        <td>The Usual Suspects</td>
        <td>1995</td>
        <td>8.5</td>
    </tr>
    </tbody>
</table>

You can also sort stuff in Thymeleaf. Given an extra model parameter passed from the servlet, and the utility object `#lists`.  
Of course, sorting is something you usually leave to your Javascript front-end (Datatables.js). But there are uses for it, and this prevents you from attaching the same list twice to the WebContext object.

```java
//java 8+ type Comparator
ctx.setVariable("movies_year_sorter", (Comparator<Movie>) (o1, o2) -> Integer.compare(o1.getYear(), o2.getYear()));

//or, a pre-java 8 Comparator
ctx.setVariable("movies_year_sorter", new Comparator<Movie>() {
    @Override
    public int compare(Movie m1, Movie m2) {
        return Integer.compare(m1.getYear(), m2.getYear());
    }
});
```

then in Thymeleaf, use the Comparator object:

```html
<tr th:each="movie:${#lists.sort(movies, movies_year_sorter)}">
```

Thymeleaf also provides a mechanism that stores the state of the iteration process. It has several useful properties:

- `index`: the current iteration index, starting with 0 (zero)  
- `count`: the number of elements processed so far  
- `size`: the total number of elements in the list  
- `even/odd`: boolean - true if the current iteration index is even or odd  
- `first`: boolean - true if the current iteration is the first one  
- `last`: boolean - true if the current iteration is the last one  

The next piece of code demonstrates the use of this iteration statistics object, as well as another technique: the if/else ternary operator for Thymeleaf. This example shows you can even nest these.

```html
<table>
    <thead>
    <tr>
        <th>Pos</th>
        <th>Title</th>
        <th>Year of release</th>
        <th>IMDB rating</th>
    </tr>
    </thead>
    <tbody>
    <tr th:each="movie, it_stat:${movies}" 
        th:class="${it_stat.first} ? 'first' : (${it_stat.even} ? 'even' : 'odd')">
        <td th:text="${it_stat.count} + ' / ' + ${it_stat.size}"></td>
        <td th:text="${movie.title}">_title_</td>
        <td th:text="${movie.year}">_year_</td>
        <td th:text="${movie.rating}">_rating_</td>
    </tr>
    </tbody>
</table>
```

when this css is applied:

```css
.even {
    background-color: aliceblue !important;
}
.odd {
    background-color: lightcyan !important;
}
.first {
    background-color: coral !important;
}
```

this is the result:

<table>
    <thead>
    <tr>
        <th>Pos</th>
        <th>Title</th>
        <th>Year of release</th>
        <th>IMDB rating</th>
    </tr>
    </thead>
    <tbody>
    <tr class="first">
        <td>1 / 8</td>
        <td>The Shawshank Redemption</td>
        <td>1994</td>
        <td>9.2</td>
    </tr>
    <tr class="even">
        <td>2 / 8</td>
        <td>The Dark Knight</td>
        <td>2008</td>
        <td>9.0</td>
    </tr>
    <tr class="odd">
        <td>3 / 8</td>
        <td>Pulp Fiction</td>
        <td>1994</td>
        <td>8.9</td>
    </tr>
    <tr class="even">
        <td>4 / 8</td>
        <td>Fight Club </td>
        <td>1999</td>
        <td>8.8</td>
    </tr>
    <tr class="odd">
        <td>5 / 8</td>
        <td>Forrest Gump</td>
        <td>1994</td>
        <td>8.7</td>
    </tr>
    <tr class="even">
        <td>6 / 8</td>
        <td>Inception</td>
        <td>2010</td>
        <td>8.7</td>
    </tr>
    <tr class="odd">
        <td>7 / 8</td>
        <td>One Flew Over the Cuckoo's Nest</td>
        <td>1975</td>
        <td>8.7</td>
    </tr>
    <tr class="even">
        <td>8 / 8</td>
        <td>The Usual Suspects</td>
        <td>1995</td>
        <td>8.5</td>
    </tr>
    </tbody>
</table>

There is a lot going on in this statement:  
`<tr th:each="movie, it_stat:${movies}"`  
`th:class="${it_stat.first} ? 'first' : (${it_stat.even} ? 'even' : 'odd')">`.
Let's dissect.  

First there is the iteration initialization where an `it_stat` object is requested as well: `th:each="movie, it_stat:${movies}"`. In the second part, a nested ternary statement sets the `class` attribute of the current row. The first level determines the first row: `th:class="${it_stat.first} ? 'first' : (<nested ternary>)"`. The nested ternary determines the class of all rows except the first: `${it_stat.even} ? 'even' : 'odd'`.

## Conditionals: if (and th:block)

Besides the ternary operator structure shown in the previous section, regular "if" tests can be applied. Here is a test for the rating of the movie; only movies with a rating higher than 9 are displayed:

```html
<ul>
    <li th:each="movie:${movies}" 
        th:text="${movie.title} + ' (' + ${movie.year} + ') - ' + ${movie.rating}" 
        th:if="${movie.rating >= 9}">_movie_</li>
</ul>
```

The specified expression evaluates to a boolean following these rules:

**_test evaluates to `true` if value is not `null` and value is_**

- a boolean and is true
- a number and is non-zero
- a character and is non-zero
- a String and is not “false”, “off” or “no”
- not a boolean, a number, a character or a String

The Thymeleaf `th:if` attribute has an inverse attribute, `th:unless`.
This makes an if/else-like structure possible. Here, you see it in combination with `th:block` which is a handy Thymeleaf element which does not generate a html DOM element. Using th:block prevents excessive use of nested div and span elements.

```html
<ul>
    <th:block th:each="movie : ${movies}">
        <li th:if="${movie.rating >= 9}" th:text="${movie.title} + ' - ' + ${movie.rating} + ' - WOW!'" ></li>
        <li th:unless="${movie.rating >= 9}" th:text="${movie.title} + ' - ' + ${movie.rating} + ' - JUST OK'" ></li>
    </th:block>
</ul>
```


## Conditionals: switch

Here you see a combination of iteration and switch/case to build an ordered list. 

```html
<ol>
    <li th:each="user:${users}" th:switch="${user.role.toString()}">
        <span th:case="${'ADMIN'}" 
            th:text="${user.name} + ' (' + ${user.role} + ') manages all accounts'">_admin_</span>
        <span th:case="${'USER'}" 
            th:text="${user.name} + ' (' + ${user.role} + ') can browse and share all site content'">_user_</span>
        <span th:case="${'GUEST'}" 
            th:text="${user.name} + ' (' + ${user.role} + ') can only see our front page'">_guest_</span>
        <span th:case="*" 
            th:text="${user.name} + '(' + ${user.role} + ') we do not know this role'">_unknown_</span>
    </li>
</ol>
```

The `th:switch` expression selects the User role (an enum) in each loop iteration, takes its String representation and compares this to a string literal in each `th:case`. Only the `th:case` that matches will be evaluated. When no match is found, the _default_ is evaluated, specified with `th:case="*"`. The result when applied to this list of User instances:

```java
List<User> users = new ArrayList<>();
User u;
u = new User("Hank", "henk@example.com", Role.ADMIN);
users.add(u);
u = new User("Roger", "roger@example.com", Role.USER);
users.add(u);
u = new User("Diana", "diana@example.com", Role.GUEST);
users.add(u);
```

is this:

```html
<ol>
    <li>
        <span>Hank (ADMIN) manages all accounts</span>
    </li>
    <li>
        <span>Roger (USER) can browse and share all site content</span>
    </li>
    <li>
        <span>Diana (GUEST) can only see our front page</span>
    </li>
</ol>
```

Note that with just one `th:case`, you can put everything within a single `li` tag - no need for the `span`.

## Working with fragments

Parts of your pages will be reused over multiple other pages. Specifically, this is the case for banners, footers, menus and headers. Instead of copy-and-paste, Thymeleaf offers a nice mechanism for reusing page fragments - even conditionally! The strategy of choice here is to create a single Thymeleaf html page called "template" or "fragments" and to include elements of this page in others.
Consider this html file called `template.html`:

```html
<!DOCTYPE html SYSTEM "http://www.thymeleaf.org/dtd/xhtml1-strict-thymeleaf-4.dtd">

<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org">
<head th:fragment="headerFragment">
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <title>_TITLE_</title>
    <link rel="stylesheet" th:href="@{/css/main.css}" href="../../css/main.css">
</head>
<body>

    <div th:fragment="banner">
        <img id="logo" th:src="@{/graphics/Awesome.png}" src="../../graphics/Awesome.png" alt="banner pic"/>
        <h1 id="banner_title">Welcome, ye of lesser quality</h1>
    </div>

    <!--these can be conditionally selected-->
    <div th:fragment="menu" class="admin">
        <a href="#">my secret content</a>
    </div>
    <div th:fragment="menu" class="normaluser">
        <a href="#">my regular content</a>
    </div>

    <div>
        <h2>Template layout</h2>
        <p>
            "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et
            dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip
            ex ea commodo consequat."
        </p>
    </div>

    <!-- receives an argument called pub -->
    <div th:fragment="footer(pub)">
        &copy; 2018 Team Awesome. Please don't contact us; we are too busy!<br/>
        But if you really need to communicate, you can find us at the local pub:
        <span th:text="${pub}">_PUB_</span><br />
    </div>

</body>
</html>
```

The different parts (divs) have been marked with `th:fragment="<fragment_name>"`. These `th:fragment` tags make them available for inclusion in other pages.

This is how you include a fragment from this page into another:

```html
<div th:insert="~{template :: banner}"></div>
<!-- shorthand -->
<div th:insert="template :: banner"></div>
```

The `th:insert` expression expects a fragment expression (`~{...}`), which is an expression that results in a fragment. The first part of this expression is the name of the html file (`template.html`) without the extension and the second part is the fragment to be included. With a non-complex fragment expression, the (`~{`,`}`) enclosing is optional, so the shorthand notation suffices.

Here is another example, the footer. This fragment takes an argument called `pub` that is available as variable by that same name. This is how to pass a variable   

```html
<div th:insert="~{template :: footer('The Bear Inn')}"></div>
```

and using it with the fragment is standard variable expression syntax: `${pub}`.

Finally, different variants of a fragment can be conditionally included - here demonstrated with a ternary statement:

```html
<div th:insert="~{template :: (${users[0].getRole().toString() == 'ADMIN'} ? 'menu.admin' : 'menu.normaluser')}"></div>
```

If the Role is ADMIN, `<div th:fragment="menu" class="admin">...` is selected, otherwise `<div th:fragment="menu" class="normaluser">...`.

### The difference between `th:insert` and `th:replace`
th:insert is the simplest: it will simply insert the specified fragment as the body of its host tag.

The `th:replace` expression actually replaces its host tag with the specified fragment.

So, this:

```html
<div th:insert="~{template :: (${users[0].getRole().toString() == 'ADMIN'} ? 'menu.admin' : 'menu.normaluser')}"></div>
```

results in 

```html
<div>
    <div class="admin">
        <a href="#">my secret content</a>
    </div>
</div>
```

whereas 

```html
<div th:replace="~{template :: (${users[0].getRole().toString() == 'ADMIN'} ? 'menu.admin' : 'menu.normaluser')}"></div>
```

results in 

```html
<div class="admin">
    <a href="#">my secret content</a>
</div>
```

