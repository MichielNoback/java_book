# Web filters and File uploads

This is the last post in this course. It deals with two loose ends: uploading files and intercepting requests.

## Web filters
Suppose you want to intercept requests for a (group of) servlets, to log what is requested and where it came from.
For instance, to do analyse the web traffic later on in order to pinpoint the most used resources.  
Or more relevant: intercept requests to asses authentication status of the user. And reject a request if the user is not authorized to see a resource.

### A basic example

Here is our old friend, the PhraseServlet (only relevant code show):

```java
package nl.bioinf.wis_on_thymeleaf.servlets;

@WebServlet(name = "PhraseServlet", urlPatterns = "/give.phrase")
public class PhraseServlet extends HttpServlet {
    private TemplateEngine templateEngine;

    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String phraseType = request.getParameter("phrase_category");
        Locale locale = request.getLocale();
        WebContext ctx = new WebContext(request, response, request.getServletContext(), locale);
        String phrase = PhraseFactory.getPhrase(phraseType);
        ctx.setVariable("phrase_type", phraseType);
        ctx.setVariable("phrase_num", phrase);
        templateEngine.process("phrase_of_the_day", ctx, response.getWriter());
    }
}
```

This is the way to keep track of requests for `/give.phrase`; a `@WebFilter` annotation on a class implementing `javax.servlet.Filter` does the trick:

```java
package nl.bioinf.wis_on_thymeleaf.webfilters;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

@WebFilter(urlPatterns = "/give.phrase")
public class PhraseServletFilter implements Filter{

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        if (request instanceof HttpServletRequest) {
            System.out.println("Request for " + ((HttpServletRequest)request).getRequestURL().toString());
            System.out.println("Query: " + ((HttpServletRequest)request).getQueryString());
            System.out.println("Remote address: " + request.getRemoteAddr());
        }
        try {
            //pass on the request after you are done
            chain.doFilter(request, response);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        /*not interesting here*/
    }

    @Override
    public void destroy() {
        /*not interesting here*/
    }
}
```

Whe the `/give.phrase?phrase_category=bullshit` resource is requested, the Tomcat logs show

<pre class="console_out">
Request for http://localhost:8080/give.phrase
Query: phrase_category=bullshit
Remote address: 0:0:0:0:0:0:0:1
</pre>

### Check for authentication

Suppose you have an application with a few public pages and a few pages for which authentication is required. This is a typical use case for WebFilter.

```java
package nl.bioinf.wis_on_thymeleaf.webfilters;


import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

@WebFilter(urlPatterns = {"/user.dashboard", "/list.secrets"}) //use "/*" for catch-all
public class AuthenticationFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        if (request instanceof HttpServletRequest) {
            HttpServletRequest req = ((HttpServletRequest)request);
            System.out.println("[AuthenticationFilter] Intercepted URL: " + req.getRequestURL().toString());
            final HttpSession session = req.getSession();
            if (session.getAttribute("user") == null) {
                System.out.println("[AuthenticationFilter] no authenticated user: redirecting to /login");
                ((HttpServletResponse)response).sendRedirect("/login");
            } else {
                System.out.println("[AuthenticationFilter] authenticated status checked; user= " + session.getAttribute("user"));
                try {
                    chain.doFilter(request, response);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        /*not interesting here*/
    }

    @Override
    public void destroy() {
        /*not interesting here*/
    }
}
```

Here is the output after a few requests

<pre class="console_out">
[AuthenticationFilter] Intercepted URL: http://localhost:8080/list.secrets 
[AuthenticationFilter] no authenticated user: redirecting to /login

&lt;logged in as Henk&gt;

[AuthenticationFilter] Intercepted URL: http://localhost:8080/list.secrets 
[AuthenticationFilter] authenticated status checked; user= User{name='Henk', email='henk@example.com', password='null', role=USER}
[AuthenticationFilter] Intercepted URL: http://localhost:8080/user.dashboard 
[AuthenticationFilter] authenticated status checked; user= User{name='Henk', email='henk@example.com', password='null', role=USER}
</pre>

## File uploads

File Upload is an essential feature! Many websites provide some form of upload functionality:
- Photos & Video
- Data
- Configuration

Fortunately, uploading is boilerplate code!

The harder thing is: what will you do with the file?
- Into database
- Store locally 
- Process directly

### Upload form

Suppose we want to offer a scv data upload.
Here is the html for uploading csv data. Key are `enctype="multipart/form-data"` in the `form` tag and the `<input type="file" name="scv_data" accept="text/csv">` tag. 

```html
<form th:action="@{'data_upload'}" method="post" enctype="multipart/form-data">
    <input type="file" name="scv_data" accept="text/csv">
    <input type="submit" value="Upload" />
</form>
```

The servlet handling the file upload needs to be annotated as such:

```java
@MultipartConfig(location="/tmp",
        fileSizeThreshold = 1024 * 1024,
        maxFileSize = 1024 * 1024 * 5,
        maxRequestSize = 1024 * 1024 * 5 * 5)
@WebServlet(name = "FileUploadServlet", urlPatterns = "/data_upload")
public class FileUploadServlet extends HttpServlet {
    //class body
}
```

The `@MultipartConfig` annotation has several parameters:

- **_location_**: An absolute path to a directory on the file system. This location is used to store files temporarily while the parts are processed or when the size of the file exceeds the specified `fileSizeThreshold` setting. The default location is "".
- **_fileSizeThreshold_**: The file size in bytes after which the file will be temporarily stored on disk. The default size is 0 bytes.
- **_maxFileSize_**: The maximum size allowed for uploaded files, in bytes. If the size is greater, the container will throw an exception (`IllegalStateExceptio`n). The default size is unlimited.
- **_maxRequestSize_**: The maximum size allowed for a multipart/form-data request, in bytes. The web container will throw an exception if the overall size of all uploaded files exceeds this threshold. The default size is unlimited.

Instead of hardcoding the constraints, you can also put them in web.xml:

```xml
    <servlet-mapping>
        <servlet-name>FileUploadServlet</servlet-name>
        <url-pattern>/data_upload</url-pattern>
    </servlet-mapping>
    <servlet>
        <servlet-name>FileUploadServlet</servlet-name>
        <servlet-class>nl.bioinf.wis_on_thymeleaf.servlets.FileUploadServlet</servlet-class>
        <init-param>
            <param-name>upload_dir</param-name>
            <param-value>/path/to/file/upload</param-value>
        </init-param>
        <multipart-config>
            <location>/path/to/tmp/tmp</location>
            <!-- all in bytes -->
            <max-file-size>20848820</max-file-size>
            <max-request-size>418018841</max-request-size>
            <file-size-threshold>1048576</file-size-threshold>
        </multipart-config>
    </servlet>
```

Here is the servlet dealing with the uploads. The final upload location is specified using an `init-param` in the `web.xml` deployment descriptor of course.

```java
package nl.bioinf.wis_on_thymeleaf.servlets;

import nl.bioinf.wis_on_thymeleaf.config.WebConfig;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.WebContext;

import javax.servlet.ServletException;
import javax.servlet.http.Part;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.IOException;

//moved to web.xml
//@MultipartConfig(location="/tmp",
//        fileSizeThreshold = 1024 * 1024,
//        maxFileSize = 1024 * 1024 * 5,
//        maxRequestSize = 1024 * 1024 * 5 * 5)
@WebServlet(name = "FileUploadServlet", urlPatterns = "/data_upload")
public class FileUploadServlet extends HttpServlet {

    private TemplateEngine templateEngine;
    private String uploadDir;

    @Override
    public void init() throws ServletException {
        this.templateEngine = WebConfig.getTemplateEngine();
        this.uploadDir = getInitParameter("upload_dir");

        //or, use relative to this app:
        // gets absolute path of the web application
        //String applicationPath = getServletContext().getRealPath("");
        //String uploadFilePath = applicationPath + File.separator + uploadDir;
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //simply go back to upload form
        WebContext ctx = new WebContext(request, response, request.getServletContext(), request.getLocale());
        templateEngine.process("upload_form", ctx, response.getWriter());
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        File fileSaveDir = new File(this.uploadDir);
        if (! fileSaveDir.exists()) {
            throw new IllegalStateException("Upload dir does not exist: " + this.uploadDir);
        }

        //Do this only if you are sure there won't be any file name conflicts!
        //An existing one will simply be overwritten
//        String fileName;
//        for (Part part : request.getParts()) {
//            fileName = part.getSubmittedFileName();
//            part.write(this.uploadDir + File.separator + fileName);
//        }

        //the safe way, with a generated file name becomes something like this
        //my_app_upload14260971264207930189.csv
        File generatedFile = File.createTempFile("my_app_upload", ".csv");
        for (Part part : request.getParts()) {
            part.write(this.uploadDir + File.separator + generatedFile.getName());
        }

        //go back to the upload form
        WebContext ctx = new WebContext(request, response, request.getServletContext(), request.getLocale());
        ctx.setVariable("message", "upload successfull, wanna do another on?");
        templateEngine.process("upload_form", ctx, response.getWriter());
    }
}
```

If you want to be sure files won't get overwritten, you should use random generated file names, as in the above code example. 

