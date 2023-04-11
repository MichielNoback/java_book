# Exercises intro

These exercises accompany the course Web-based Information Systems. After the first exercise, we will work on a single project that we will improve and extend iteratively as new information becomes available.

----

## Implementing MVC 

**Goal:** Create a web application that gives the heart rate zones based on the maximum heart rate.

### Background

The maximum heart rate can be determined by running (preferably on a treadmill but outside will do) as fast as you can for 3 minutes straight, followed by a three-minute gentle run and again 3 minutes full force. Optionally, add another cycle.
The maximum reading is the maximum heart rate.

The heart rate zones can be calculated by this rule set, giving zone boundaries and descriptions as % of maximum heart rate: 
- zone 1: 50-60% Low heart rate with hardly any training effect
- zone 2: 60-70% Heart rate with positive effect on breathing and stress reduction
- zone 3: 70-80% Heart rate with optimal training effect with respect to fat burning, general condition and brain functioning
- zone 4: 80-90% Zone for improving (long-distance) running speed
- zone 5: 90-100% Above anaerobic threshold; only during interval training to increase speed (also for shorter distances).

### The exercise

All texts in this app should be located in the resource bundle(s), not in the html and neither in the Java classes.

Create appropriate packages for your Java classes.

1. Start by creating a new Gradle-managed web project.  

2. Create the servlet dealing with the url `/heart-rate-zones`. Its `doGet()` method should forward to a Thymeleaf html page.

3. In this Thymeleaf page, create a form asking the user for her maximum heart rate (and giving usage instructions). It should support both Dutch and English. The form should direct back to the servlet serving url `/heart-rate-zones`, but this time to the `doPost()` method. 

4. In the `doPost()` method, the servlet should invoke a model class responsible for calculating the correct heart rate zones.

5. These zones should be displayed in another html Thymeleaf view, again supporting the languages Dutch and English.

Note: yes this could have been done more efficiently using Javascript, but we are working with Java for now.


----

## Thymeleaf

**Goal:** Create a "SpeciesBrowser" web application that publishes information on some species from your favorite animal or plant group.

### Background
This assignment will encourage you to use as many Thymeleaf expressions as possible, as well as resource bundles.  

As preparation, you should choose an animal or plant group that you think is interesting. From this group of organisms, 
choose at least 5 representatives and gather some relevant information, as well as one picture for each species.
The information should include at least:
- Scientific name
- English name
- Dutch name

Here is an example of one of my favorite birds, the steppe eagle (copied from Wikipedia):


**Steppe eagle (_Aquila nipalensis_)**  
**Dutch name**: Steppearend

![Steppe eagle](figures/steppe_eagle.jpg)
(By T. R. Shankar Raman - <span class="int-own-work" lang="en">Own work</span>, <a href="https://creativecommons.org/licenses/by-sa/3.0" title="Creative Commons Attribution-Share Alike 3.0">CC BY-SA 3.0</a>, <a href="https://commons.wikimedia.org/w/index.php?curid=60338791">Link</a>)

**Description**  
The steppe eagle (_Aquila nipalensis_) is a bird of prey. Like all eagles, it belongs to the family _Accipitridae_. It was once considered to be closely related to the non-migratory tawny eagle (Aquila rapax) and the two forms have previously been treated as conspecific. They were split based on pronounced differences in morphology and anatomy.
Wing span: 1.65–2.15 meters  
**Occurrence**  
It is a migratory bird breeding in eastern Europe, the Middle East and Asia and wintering in Africa and South Asia.  
**Conservation status**  
Threatened (declining populations)

Note, you are also allowed to gather "famous biologists".

### The exercise

1. Start by creating a new Gradle-managed web project. Add git versioning support and create a remote (Bitbucket or Github). 

2. Create a simple "csv" file in a data folder that holds, for each species, all non-internationalized textual data. Here is my Steppe Eagle example:

    ```
    scientific_name;english_name;dutch_name;wing_span_avg;picture_loc
    Aquila nipalensis;Steppe Eagle;steppearend;1.9;figures/steppe_eagle.jpg
    ```

3. Collect all other (textual) data in a resource bundle, supporting both English and Dutch. This concerns primarily text labels and page content, but also species descriptions.

4. Design and implement Java classes that model your information. In this case, class `Raptor` or `Bird` would be nice. Also, think about the way your objects are going to be managed. Should they be retrievable by name or something? Create a "CollectionClass" that supports this usage.

5. Create a class that can parse your csv file into a "CollectionClass", holding a collection of instances representing your species.

6. Create a servlet serving url `/home` that redirects to a Thymeleaf page called `species-listing.html`. 
This page should list all available species in a table, providing for each species a hyperlink to a species-detail page (look at the html cheat sheet how to do this). The link should point to url `/species.detail`. This is a Thymeleaf example:

    ```html
    <a th:href="@{'/species.detail?species=' + ${species.name}}" th:text="${species.name}"></a>
    ```
    This page should be internationalized for Dutch and English of course.

7. Create a servlet serving url `/species.detail` that directs to a Thymeleaf page showing the detailed information of a selected species (linked via the table created before), as well as a nice picture of course.
This page is also internationalized.

8. **_Tag_** the final version of the repo as 0.1.0. and commit this to your remote.

9. **_[Challenge]_** Use Bootstrap to make the two pages look nice as well, or apply your own styling.

You are of course free to extend the information with whatever you think is nice.  

----

## Sessions 

**Goal:** Modify a web application to support browsing history.

### Background
The web app that was developed in the previous exercise is very static and impersonal. We'll add some functionality to fix that.

What we'll do:
- add browsing history support
- put this history in an include that is shown on both pages

### The exercise

First open the "SpeciesBrowser" web app project of the previous exercise.

0. I don't know where you put your data file location in your code, but it is time to move this to `web.xml`.

1. Modify the servlet serving `/home` so that it will create a session object when first requested by a user.

2. Create an appropriate data structure (read: class) to hold browsing history of only the last 5 page views, and attach this to the user's session object.

3. Whenever the user views the details page of a species, add this species to the history.

4. Create an included `div` fragment in a file called `template.html` that will display the history listing.

5. Show the history listing by including the div as a separate panel in both your Thymeleaf templates (the listing and the details page). 

6. **_Tag_** the final version of the repo as 0.2.0. and commit this to your remote.

----- 

## Authentication

### Background
We continue working on the "SpeciesBrowser" web app.

What we'll do:
- add login functionality
- provide a web form to edit species data for registered users

### The exercise

1. Create a `User` class that will model your web app users.

2. Create a "mock" database containing 3 users.

3. Create a login page that directs to a servlet with url mapping `/login`.

4. Create the login servlet. Verify the user credentials. If correct, redirect to `/home` (the species listing). Else, redirect back to `login.html`, giving the user a warning that the provided credentials were incorrect.

5. In the species detail page, add a button -only when there is an authorized user- that will open a page in which the species information can be edited. Note: editing information that is contained within resource bundles is not possible! Choose other information fields.

6. Create the editing form.

8. **_Tag_** the final version of the repo as 0.3.0. and commit this to your remote.

9. **_[Challenge 1]_** Process the edited data coming from the above form and add support for writing species data to file.

10. **_[Challenge 2]_** Add registration functionality.

----

## Servlet details and hooks

We continue working on the "SpeciesBrowser" web app.

**Goal:** Further refactoring to improve performance and maintainability.

### Background

We'll work at optimizing the app. So far, you probably have loaded the entire data file at each request. This is of course highly inefficient. Let's move this to a `@WebListener` method.

### The exercise

1. Modify your "CollectionClass" so that it will serve your species collection in a static manner. This means the method should be class level. 

2. Implement the `ServletContextListener` interface and add the `@WebListener` annotation. In the `contextInitialized()` method, have the "CollectionClass" load the species data collection.

3. Refactor the rest of the app to only use this static method.

4. **_Tag_** the final version of the repo as 0.4.0. and commit this to your remote.

----

## Database interaction

We continue working on the "SpeciesBrowser" web app.

**Goal:** Add a replaceable MySQL data layer.

### Background

We'll work at extending the app. The species information located in the csv file will be migrated to a MySQL data layer and abstracted away behind an interface.

As a prerequisite, it is assumed you have a MySQL database server and -account.

### The exercise

1. Add support for MySQL/JDBC in your project.

2. Design the database table(s) and create them.

3. Design an interface declaring the public contract for your species information datasource, whatever the nature of that datasource.

4. Have your original "CollectionClass" implement this interface.

5. Make a MySQL implementer of the same interface. Implement it using the Singleton Pattern.  
(**_[Challenge]_** make it thread-safe)

6. Create a factory class that serves the correct implementer based on a setting in the `web.xml` file.

7. Refactor your code base to only "talk to" the interface served by the factory class.

8. **_[Challenge]_** Also migrate the user data (and species detail information) to this database.

9. **_Tag_** the final version of the repo as 0.5.0. and commit this to your remote.

----

## Ajax

We continue working on the "SpeciesBrowser" web app.

**Goal:** Add an Ajax data service.

### Background

In this exercise, you will create a "microservice" that will serve species details to be used as a remote API in other apps.

### The exercise

1. Add support for Googles' Gson library to your Gradle file.

2. Create a servlet that maps to `/species_info`. The servlet should only support `GET` requests. As request parameters, accept `species=<name_or_substring>` and `info=<short|long>`, where `short` will only return the species names and "short" data fields that are present in the database of those species where (one of) the name(s) match the given string.  
**_[Challenge]_** The `long` option will return the species description. You should figure out how to read this from the resource bundle.

3. Design an appropriate response if  
    * the data layer is unresponsive, 
    * there were unsupported parameters provided or  
    * there is no species that matches. 

4. Test it out using `curl` and your web browser.

5. **_Tag_** the final version of the repo as 0.6.0. and commit this to your remote.

----

## Filters

We continue working on the "SpeciesBrowser" web app.

**Goal:** Add extra security using filters.

### Background

In this exercise, you will finish the app by creating a Filter that intercepts requests for the species editing form.

### The exercise

1. Create a `@WebFilter` implementing the `Filter` interface so that requests for the species editing form are intercepted and only pass for logged-in users. When the request does not come from a logged-in user, redirect back to `/login`. 

2. Remove the redundant code from the servlet.

3. **_Tag_** the final version of the repo as 1.0.0. with name "Release Candidate" and commit this to your remote.
