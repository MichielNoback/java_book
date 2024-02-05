# Demo case refactoring

## Introduction  

This will be quite a long post. One I hope you will finish! If you want to follow along in the code base, 
you should clone the repo. It is available at [bitbucket](https://michiel_noback@bitbucket.org/minoba/refactoringtrainingproject.git).
Check out commit `eaf007f` to have a look at the original project. It was created by a student of mine, a third year bioinformatics student who will remain anonymous. As final assignment of the course Introduction to Java programming, he wrote a 
tool that can be used to read, summarize and filter files of the `gff3` format (see[https://www.ensembl.org/info/website/upload/gff3.html](https://www.ensembl.org/info/website/upload/gff3.html)). 
This flat-file format is used to describe (annotate) elements (features), such as genes and introns, on DNA molecules, 
also supporting hierarchical or tree-like relationships. Here is a small example, copied (and slimmed down) from a much
 larger file describing some piece of Potato DNA:

```
PGSC0003DMB000000010	BGI	gene	1242814	1245448	.	.	.	ID=PGSC0003DMG400000001;
PGSC0003DMB000000010	Cufflinks	mRNA	1242814	1245444	.	+	.	ID=PGSC0003DMT400000001;Parent=PGSC0003DMG400000001;Source_id=RNASEQ44.3678.0;Mapping_depth=48.922289;Class=1;
PGSC0003DMB000000010	Cufflinks	mRNA	1242814	1245444	.	+	.	ID=PGSC0003DMT400000002;Parent=PGSC0003DMG400000001;Source_id=RNASEQ44.3678.1;Mapping_depth=349.773769;Class=1;
PGSC0003DMB000000010	Cufflinks	mRNA	1243971	1245448	.	+	.	ID=PGSC0003DMT400000003;Parent=PGSC0003DMG400000001;Source_id=RNASEQ44.3678.2;Mapping_depth=52.447281;Class=1;
PGSC0003DMB000000010	Cufflinks	exon	1242814	1243429	.	+	.	ID=PGSC0003DME400000001;Parent=PGSC0003DMT400000001,PGSC0003DMT400000002;
PGSC0003DMB000000010	Cufflinks	exon	1243532	1243736	.	+	.	ID=PGSC0003DME400000002;Parent=PGSC0003DMT400000001,PGSC0003DMT400000002;
PGSC0003DMB000000010	Cufflinks	intron	1243430	1243531	.	+	.	ID=PGSC0003DMI400000001;Parent=PGSC0003DMT400000001,PGSC0003DMT400000002;
PGSC0003DMB000000010	Cufflinks	intron	1244851	1244983	.	+	.	ID=PGSC0003DMI400000005;Parent=PGSC0003DMT400000002;
PGSC0003DMB000000010	BestORF	CDS	1243614	1243736	.	+	0	ID=PGSC0003DMC400000001;Parent=PGSC0003DMT400000001;
PGSC0003DMB000000010	BestORF	CDS	1244206	1244487	.	+	0	ID=PGSC0003DMC400000001;Parent=PGSC0003DMT400000001;
PGSC0003DMB000000155	BGI	gene	1093109	1102125	.	.	.	ID=PGSC0003DMG400000278;
```

It is a tab-separated file with this -abbreviated- specification:

Fields must be tab-separated. Also, all but the final field in each feature line must contain a value; "empty" columns should be denoted with a '.'  

1. **seqid** - name/ID of the chromosome or segment
2. **source** - name of the program that generated this feature
3. **type** - type of feature. 
4. **start** - Start position of the feature, starting at 1.
5. **end** - End position of the feature, starting at 1.
6. **score** - A floating point value.
7. **strand** - defined as + (forward) or - (reverse).
8. **phase** - One of '0', '1' or '2'. '0' indicates that the first base of the feature is the first base of a codon, '1' that the second 
base is the first base of a codon, and so on.
9. **attributes** - A semicolon-separated list of tag=value pairs, providing additional information about each feature. Some of these tags 
are predefined, e.g. ID, Name, Alias, Parent.  

I chose this particular project because (a) it is simple enough to be understood (b) complex enough to be a serious challenge (c) it had 
so many inline comments that the unintelligible code could be understood and (d) the functionality was (as far as I tested) not perfect, 
but OK. The student had implemented all the required use cases. But the code itself was written like a big old script, with many (naming) 
convention violations, OO violations, Encapsulation and Abstraction not applied. You'll see many examples as we plod along. 

I do not plan to finish the whole project here; there should be some challenge for you left.

### Note on JUnit tests 

When taking over a code base like this, without any (J)Unit tests, it is a very good idea to develop JUnit tests in concert with your refactoring efforts.
Take a look at the (too) few JUnit tests that I added to this project as well.


## Get to know what is there  

First, of course, you need to run the tool in different modes and study the existing code base to what exactly is going on.  

Running it in this mode `-i data/gff3_sample.gff3 -s` will give a summary:

```
file	gff3_sample.gff3
total number of features	22
molecules with features:
human15.1
	number of features	22
		CDS				10
		gene				1
		three_prime_UT				4
		mRNA				4
		five_prime_UTR				3
```

That's not really lined out but it'll do for now.

Another use case, looking for features of a specific type `-i data/gff3_sample.gff3 -ft mRNA`:

```
human15.1 . mRNA            214360  215771 . +   . Comments=fixed+one+splice+junction;Parent=HsG8283;Evidence=7000000069743825;Transcript_type=Novel_Transcript;Name=Novel+Transcript%2C+variant+%28partial%29;ID=HsT20206
human15.1 . mRNA            214590  215772 . +   . Comments=fixed+one+splice+site%0A;Parent=HsG8283;Evidence=7000000069600840;Transcript_type=Novel_Transcript;Name=Novel+Transcript%2C+variant+%28partial%29;ID=HsT20207
human15.1 . mRNA            214301  215769 . +   . Parent=HsG8283;Evidence=7000000069974357;Transcript_type=Candidates+for+Deletion;Name=Novel+Transcript+%28partial%29;ID=HsT16028
human15.1 . mRNA            215218  215772 . +   . Parent=HsG8283;Evidence=7000000069512231;Transcript_type=Novel_Transcript;Name=Novel+Transcript%2C+variant;ID=HsT16029
```

Some more use cases were carried out. There do not seem to major bugs.

This is the simple class and package layout of the project:

```
gff_query/control
            GffLine
         /io
            CLIParser
            FileReader
         /model
            GffFile
         GffQuery (main)
```

This puzzles me a bit. I would expect something like GffLine to be in the `model` and `CLIParser` somewhere not in the `io` package. We'll get to that.

A first inspection of the classes is a bit of a scare. Here is class `FileReader`. 

```java
package nl.bioinf.gff_query.io;
//imports omitted
public class FileReader {
    public ArrayList<String> readFile(String raw_path) {
        ArrayList<String> fileArrayList = new ArrayList<>();
        Path path = Paths.get(raw_path);
        try (BufferedReader reader = Files.newBufferedReader(
                path,
                Charset.defaultCharset())) {
            String lineFromFile;
            while ((lineFromFile = reader.readLine()) != null) {
                fileArrayList.add(lineFromFile);
            }
        } catch (IOException exception) {
            System.out.println("Error while reading file, \"IOExceptio\", Did you give the correct location / does the " +
                "file exist?");
        }
        return fileArrayList;
    }
}
```

OK...  

This class does nothing more than processing the input file into a list of lines. Hello! Streaming processing? These files can get in the Gigabytes size.
Moreover, it catches an exception only to generate a `System.out.println` message. Some work to be done here for sure. Note the name `raw_path`. This is somebody coming from Python, it seems.

Next class, main: `GffQuery`. Take a minute to study its source before reading on. Note at least 5 points of improvement elegible for refactoring.

```java
package nl.bioinf.gff_query;
//imports omitted

public class GffQuery {
    public static void main(String[] args) throws IOException {
        // grab stuff from command line, try to parse.
        CLIParser my_cli_parser = new CLIParser();
        // try to parse
        try {
            // before anything else, try for printing the help
            boolean help_present = my_cli_parser.cliHelpParse(args);
            if (help_present) {my_cli_parser.printHelp();}
            else{
                // parse the command line arguments
                my_cli_parser.cliParse(args);
                // if this worked, go on and grab all the arguments.
                // but first check if the inputs are correct.
                boolean correct_inputs = my_cli_parser.checkInputs();
                if (!correct_inputs) {
                    System.out.println("Something went wrong when checking inputs, some may be incorrect, please " +
                            "refer to the help page or check if your input file is correct.");
                }
                else {
                    String[] parsed_arguments = my_cli_parser.returnArguments();
                    // handle stuff based on the case.
                    int mycase;
                    if (parsed_arguments[1].equals("true")) {
                        // case 1, print summary.
                        mycase = 1;
                    }
                    else {
                        // case 2, actually apply filters and stuff.
                        mycase = 2;
                    }
                    // before running the switch, read in file and make a GffFile object
                    String path = parsed_arguments[0];
                    FileReader filereader = new FileReader();
                    // read in the raw file and catch the resulting ArrayList
                    ArrayList fileArrayList = filereader.readFile(path);
                    if (fileArrayList.isEmpty()) {
                        mycase = 3;
                    }
                    // parse it to a GffFile object.
                    GffFile myGffFile = new GffFile(fileArrayList, path);
                    switch (mycase) {
                        case 1:
                            // do summary, not implemented as of yet.
                            myGffFile.printSummary();
                            break;
                        case 2:
                            // apply all filters.
                            // create filters array
                            String[] filters = new String[5];
                            // filters has all filters like so;
                            // fetch_type, fetch_region, filter, fetch_children, find_wildcard
                            // parsed_arguments had this;
                            // infile, summary, fetch_type, fetch_region, fetch_children, find_wildcard, filter
                            filters[0] = parsed_arguments[2];
                            filters[1] = parsed_arguments[3];
                            filters[2] = parsed_arguments[6];
                            filters[3] = parsed_arguments[4];
                            filters[4] = parsed_arguments[5];
                            // put in filters and print results
                            myGffFile.applyFilters(filters);
                            if (myGffFile.toString().isEmpty()) {
                                System.out.println("Oops, no lines found! Try to apply different filters. or double " +
                                        "check them. It might also be possible that the file did not contain any" +
                                        " valid gff3 formatted lines.");
                            }
                            System.out.print(myGffFile.toString());
                            break;
                        case 3:
                            System.out.println("empty or corrupt file.");
                            break;
                    }

                }
            }
        }
        catch( ParseException exp ) {
            // oops, something went wrong
            System.err.println( "command line input parsing failed. please refer to the manual using the \"--help\" " +
                    "argument" );
        }
    }
}
```

OK...again...

So here are a few aspects that need to be addressed:  

1. Every line of code is in one big main
2. This is a Many Responsibilities class
3. Naming convention violations all over
4. Too complex flow control
5. Not OO at all
6. and more

We'll get to this class later.

The `GffLine`, `GffFile` and `CLIParser` classes are similar in nature. I won't show them all here. In this part, I would like to focus on class `CLIParser`. To get it right, I will need to refactor in other places as well, but this will be my starting point. Why start here? This is a class that should have an obvious Single Responsibility: taking care of command-line arguments. Have a look at the code yourself first and think where you would start.  

```java
package nl.bioinf.gff_query.io;

import org.apache.commons.cli.*;
import java.io.File;

public class CLIParser {
    private Options options;
    private Options help_options;
    private CommandLine cmd;
    private CommandLine help_cmd;
    private DefaultParser parser;
    private HelpFormatter formatter;

    public CLIParser() {
        // Create parser object
        this.parser = new DefaultParser();
        // create Options object
        this.options = new Options();
        this.help_options = new Options();
        // add cli options

        Option helpOption = Option.builder("h")
                .longOpt("help")
                .required(false)
                .hasArg(false)
                .desc("Gives program help.")
                .build();
        this.help_options.addOption(helpOption);
        this.options.addOption(helpOption);
        Option infileOption = Option.builder("i")
                .longOpt("infile")
                .required(true)
                .hasArg(true)
                .desc("input gff3 file, e.g. \"gff3_sample.gff3\"")
                .build();
        this.options.addOption(infileOption);
        Option summaryOption = Option.builder("s")
                .longOpt("summary")
                .required(false)
                .hasArg(false)
                .desc("makes a summary of the gff3 file.")
                .build();
        this.options.addOption(summaryOption);
        Option fetchtypeOption = Option.builder("ft")
                .longOpt("fetch_type")
                .required(false)
                .hasArg(true)
                .desc("fetches type, e.g. \"CDS\"")
                .build();
        this.options.addOption(fetchtypeOption);
        Option fetchregionOption = Option.builder("fr")
                .longOpt("fetch_region")
                .required(false)
                .hasArg(true)
                .desc("fetches region in between start..end, e.g. \"250000..260000\"")
                .build();
        this.options.addOption(fetchregionOption);
        Option fetchchildrenOption = Option.builder("fc")
                .longOpt("fetch_children")
                .required(false)
                .hasArg(true)
                .desc("input which Parent group attributes to sort on, e.g. \"PGSC0003DMT400039136\".")
                .build();
        this.options.addOption(fetchchildrenOption);
        Option findwildcardOption = Option.builder("fw")
                .longOpt("find_wildcard")
                .required(false)
                .hasArg(true)
                .desc("input which wildcard, e.g. \"[dD]efesin\" to match on. (gff3 name attribute)")
                .build();
        this.options.addOption(findwildcardOption);
        Option filterOption = Option.builder("f")
                .longOpt("filter")
                .required(false)
                .hasArg(true)
                .desc("miscellaneous filters, defined like" +
                        " so: \"source|score|orientation|maximum_length|minimum_length\"")
                .build();
        this.options.addOption(filterOption);
        // create help
        this.formatter = new HelpFormatter();
    }

    public void cliParse(String[] args) throws ParseException {
        this.cmd = this.parser.parse(this.options, args);
    }

    public boolean checkInputs() {
        // check which inputs exist and check them (if neccesary)
        // check infile
        if (this.cmd.hasOption("infile")) {
            File f = new File(this.cmd.getOptionValue("infile"));
            if(!(f.exists() && !f.isDirectory())) {
                // raise error that file does not exist.
                return false;
            }
        }
        // check filter
        if (this.cmd.hasOption("filter")) {
            // check if filter has correct formatting. If it does not, return false
            // <SOURCE, SCORE, ORIENTATION MAXIMUM AND/OR MINIMUM LENGTH>
            if (!this.cmd.getOptionValue("filter").matches(
                    "(.+\\|(\\d+|\\*|\\.)\\|(\\.|\\+|\\-|\\*)\\|(\\d+|\\*|\\.)\\|(\\d+|\\*|\\.))")) {
                return false;
            }
        }
        // check fetch region
        if (this.cmd.hasOption("fetch_region")) {
            // check if fetch region has correct formatting.
            if (!this.cmd.getOptionValue("fetch_region").matches("\\d+\\.\\.\\d+")){
                return false;
            }
        }

        if (this.cmd.hasOption("fetch_type")) {
            // check if not a bool.
            if (this.cmd.getOptionValue("fetch_type").matches("(true|false)")) {
                return false;
            }
        }
        if (this.cmd.hasOption("fetch_children")) {
            // check if not a bool.
            if (this.cmd.getOptionValue("fetch_children").matches("(true|false)")) {
                return false;
            }
        }
        if (this.cmd.hasOption("find_wildcard")) {
            // check if not a bool.
            if (this.cmd.getOptionValue("find_wildcard").matches("(true|false)")) {
                return false;
            }
        }
        return true;
    }

    public String[] returnArguments() {
        String[] parsed_args = new String[7];
        // build parsed_args array.
        // it is filled with options/null respectively like so;
        // infile, summary, fetch_type, fetch_region, fetch_children, find_wildcard, filter
        // Options without arguments, will be false if not present, and true when present.
        // Options with arguments, will also be false if not present, and their getOptionValue when present.
        // keep all values strings to be consistent.
        if (this.cmd.hasOption("infile")) {parsed_args[0] = this.cmd.getOptionValue("infile");}
        else {parsed_args[0] = "false";}
        if (this.cmd.hasOption("summary")) {parsed_args[1] = "true";}
        else {parsed_args[1] = "false";}
        if (this.cmd.hasOption("fetch_type")) {parsed_args[2] = this.cmd.getOptionValue("fetch_type");}
        else {parsed_args[2] = "false";}
        if (this.cmd.hasOption("fetch_region")) {parsed_args[3] = this.cmd.getOptionValue("fetch_region");}
        else {parsed_args[3] = "false";}
        if (this.cmd.hasOption("fetch_children")) {parsed_args[4] = this.cmd.getOptionValue("fetch_children");}
        else {parsed_args[4] = "false";}
        if (this.cmd.hasOption("find_wildcard")) {parsed_args[5] = this.cmd.getOptionValue("find_wildcard");}
        else {parsed_args[5] = "false";}
        if (this.cmd.hasOption("filter")) {parsed_args[6] = this.cmd.getOptionValue("filter");}
        else {parsed_args[6] = "false";}

        return parsed_args;
    }

    public boolean cliHelpParse(String[] args) throws ParseException {
        this.help_cmd = new DefaultParser().parse(this.help_options, args, true);
        if(this.help_cmd.hasOption("help") || this.help_cmd.getArgs().length == 0) {
            return true;
        }
        return false;
    }

    public void printHelp() {
        // print the help or the version there. return true because help is printed
        String footer = "//omitted for brievity//";

        this.formatter.printHelp("GffQuery", "Version: 1.1-SNAPSHOT", this.options, footer, true);
    }
}
```

Since I really -really- hate it when people don't adhere to naming conventions, I'll start there.

## Refactor

### Rename variables and methods to adhere to Java standards. 

So, `help_options` becomes `helpOptions`, and `help_cmd` becomes `helpCommandLine`. You may 
note the absence of `help_cmd` in the final version. This is because there is no reason to maintain
 it as an instance variable.

Remember by the way that methods should be verbs and variables should be nouns.

### Extract string literal to constants 

As an example, you see the word "infile" typed several times: 
`Option infileOption = Option.builder("i").longOpt("infile")`, `if (this.cmd.hasOption("infile"))` 
(2 times). So this is the next step: get these out of the real code.

```java
public class CLIParser {
    //much code omitted in this snippet!
    public static final String OPTION_INFILE = "infile";
    public static final String OPTION_SUMMARY = "summary";
    //...

    public CLIParser() {
        Option infileOption = Option.builder("i")
                .longOpt(OPTION_INFILE)
                .required(true)
                .hasArg(true)
                .desc("input gff3 file, e.g. \"gff3_sample.gff3\"")
                .build();
        //...
    }
    public boolean checkInputs() {
        // check which inputs exist and check them (if neccesary)
        // check infile
        if (this.cmd.hasOption(OPTION_INFILE)) {
            File f = new File(this.cmd.getOptionValue("infile"));
            if(!(f.exists() && !f.isDirectory())) {
                // raise error that file does not exist.
                return false;
            }
        }
        //...
    }
}
```

Should the constants be public? Maybe, maybe not. Think about it.

### Extract functionality from constructor code. 

Constructors should be short and readable, and only construct (maybe through calling other methods).
What I sometimes even see is code like this:

```java
public class MyApp{
    public static void main(String[] args) {
        MyApp(args);
    }
    //more code
}
```

That means there is application logic in the constructor that is not even called explicitly!

So here is the new constructor of the CLIParser class:

```java
    public CLIParser() {
        this.parser = new DefaultParser();
        buildOptions();
    }
```

It calls the `buildOptions()` method to, well, build te options. This method creates both `this.options = new Options();` 
and `this.helpOptions = new Options();`. That seems a bit funny but this is a workaround for the problem with `infileOption`
being required. This will throw an exception if it is absent. That conflicts with the case where only help is requested.

### Rename methods and variables to their real purpose. 

The method `cliHelpParse()` does not parse help. No, its intent is to signal that help is requested:

```java
    public boolean cliHelpParse(String[] args) throws ParseException {
        this.help_cmd = new DefaultParser().parse(this.help_options, args, true);
        if(this.help_cmd.hasOption("help") || this.help_cmd.getArgs().length == 0) {
            return true;
        }
        return false;
    }
```

So I refactored it into the following version, with a more suitable name, the commandLine variable now being local, and the
 `ParseException` not thrown anymore (this choice can be argued extensively).

```java
    public boolean isHelpRequested(String[] args) {
        try {
            CommandLine commandLine = this.parser.parse(this.helpOptions, args);
            return (commandLine.hasOption(OPTION_HELP));
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return true;
    }
```

A variant to this solution could be:

```java
    public boolean isHelpRequested(String[] args) {
        try {
            CommandLine commandLine = this.parser.parse(this.options, args);
            return (commandLine.hasOption(OPTION_HELP));
        } catch (ParseException e) {
            return true;
        }
    }
```

Here, the helpOption instance variable has been made obsolete, and in the catch block is is simply assumed that if 
anything goes wrong, help should be displayed. Which is the better?  

Next, method `public void cliParse(String[] args)` was renamed to `public void parseCommandLineArguments(String[] args)`.

### The Single Responsibility Principle  

The next aspect is the method `public boolean checkInputs() {}`. Is class CLIParse the place to do this? What if some other 
means of options specification is implemeted, e.g. a GUI form? These options will need to be checked all the same. I think 
`checkInputs` should not be in that class. But where else? Main? No, I think checking options, and keeping track of options, 
should be the responsibility of a separate class. So that is where they'll go. I created class `GffAnalysisOptions` and put
 option checking in it:

```java
public class GffAnalysisOptions {
    private String inFile;
    private boolean summary;
    private String searchType;
    private String searchRegion;
    private String searchChildren;
    private String searchWildcard;
    private String searchFilter;

    //getters and setters

    public boolean checkCorrectnessOfInputs() {
        //checking code 
    }

}
```

### Use basic OOP principles 

So, finally, there is this method (abbreviated):

```java
    public String[] returnArguments() {
        String[] parsed_args = new String[7];
        // build parsed_args array.
        // it is filled with options/null respectively like so;
        // infile, summary, fetch_type, fetch_region, fetch_children, find_wildcard, filter
        // Options without arguments, will be false if not present, and true when present.
        // Options with arguments, will also be false if not present, and their getOptionValue when present.
        // keep all values strings to be consistent.
        if (this.commandLine.hasOption(OPTION_INFILE)) {parsed_args[0] = this.commandLine.getOptionValue("infile");}
        else {parsed_args[0] = "false";}
        if (this.commandLine.hasOption(OPTION_SUMMARY)) {parsed_args[1] = "true";}
        else {parsed_args[1] = "false";}
        if (this.commandLine.hasOption(OPTION_FETCH_TYPE)) {parsed_args[2] = this.commandLine.getOptionValue("fetch_type");}
        else {parsed_args[2] = "false";}
        if (this.commandLine.hasOption(OPTION_FETCH_REGION)) {parsed_args[3] = this.commandLine.getOptionValue("fetch_region");}
        else {parsed_args[3] = "false";}
        if (this.commandLine.hasOption(OPTION_FETCH_REGION)) {parsed_args[4] = this.commandLine.getOptionValue("fetch_children");}
        else {parsed_args[4] = "false";}
        if (this.commandLine.hasOption(OPTION_FIND_WILDCARD)) {parsed_args[5] = this.commandLine.getOptionValue("find_wildcard");}
        else {parsed_args[5] = "false";}
        if (this.commandLine.hasOption(OPTION_FILTER)) {parsed_args[6] = this.commandLine.getOptionValue("filter");}
        else {parsed_args[6] = "false";}

        return parsed_args;
    }
```

Auch! This hurts. A String[] with "false" values stored in it. Moreover, the code is so obfuscated that it needs many lines of 
comments. Why didn't this student think of an Options class of some kind, like `GffAnalysisOptions` (shown above)?  

So here I wrote a replacement method for it:

```java
    public GffAnalysisOptions getAnalysisOptions() {
        GffAnalysisOptions gffAnalysisOptions = new GffAnalysisOptions();
        if (this.commandLine.hasOption(OPTION_INFILE)) {
            gffAnalysisOptions.setInFile(this.commandLine.getOptionValue(OPTION_INFILE));
        }
        if (this.commandLine.hasOption(OPTION_SUMMARY)) {
            gffAnalysisOptions.setSummary(true);
        }
        if (this.commandLine.hasOption(OPTION_FETCH_TYPE)) {
            gffAnalysisOptions.setSearchType(this.commandLine.getOptionValue(OPTION_FETCH_TYPE));
        }
        if (this.commandLine.hasOption(OPTION_FETCH_REGION)) {
            gffAnalysisOptions.setSearchRegion(this.commandLine.getOptionValue(OPTION_FETCH_REGION));
        }
        if (this.commandLine.hasOption(OPTION_FETCH_CHILDREN)) {
            gffAnalysisOptions.setSearchChildren(this.commandLine.getOptionValue(OPTION_FETCH_CHILDREN));
        }
        if (this.commandLine.hasOption(OPTION_FIND_WILDCARD)) {
            gffAnalysisOptions.setSearchWildcard(this.commandLine.getOptionValue(OPTION_FIND_WILDCARD));
        }
        if (this.commandLine.hasOption(OPTION_FILTER)) {
            gffAnalysisOptions.setSearchFilterAsString(this.commandLine.getOptionValue(OPTION_FILTER));
        }
        return gffAnalysisOptions;
    }
```

Of course, these options need to be processed further - primarily the filter option - before they can be put to use.

To replace the (fortunately localized) usage of the `String[]` with an `GffAnalysisOptions` instance, I switched to 
class `GffQuery` and eased it in:

```java
    public static void main(String[] args) throws IOException {
        //much more code

        String[] parsedArguments = cliParser.returnArguments();
        GffAnalysisOptions analysisOptions = cliParser.getAnalysisOptions();
        //TODO replace use of parsedArguments with analysisOptions

        //more code

        // handle stuff based on the case.
        int mycase;
        if (parsedArguments[1].equals("true")) {
            // case 1, print summary.
            mycase = 1;
        }
        else {
            // case 2, actually apply filters and stuff.
            mycase = 2;
        }

        //much more code, also like this:
            // infile, summary, fetch_type, fetch_region, fetch_children, find_wildcard, filter
            filters[0] = parsedArguments[2];
            filters[1] = parsedArguments[3];
            filters[2] = parsedArguments[6];
            filters[3] = parsedArguments[4];
            filters[4] = parsedArguments[5];
    }
```

It is extremely tempting to refactor the other ugly code as well, such as the `mycase = 1` ot the `String[] filters`, but restraint is very important in this kind of operation.
So, only refactor out the use of `parsedArguments` now, and keep on writing and running your JUnit tests to verify no functionality gets broken:

```java
            if (analysisOptions.isSummaryRequested()) {
//            if (parsedArguments[1].equals("true")) {
                // case 1, print summary.
                mycase = 1;
            }
            else {
                // case 2, actually apply filters and stuff.
                mycase = 2;
            }
            // before running the switch, read in file and make a GffFile object
            String path = analysisOptions.getInFile();//parsedArguments[0];

            //more code 

                    String[] filters = new String[5];
                    // filters has all filters like so;
                    // fetch_type, fetch_region, filter, fetch_children, find_wildcard
                    // parsedArguments had this;
                    // infile, summary, fetch_type, fetch_region, fetch_children, find_wildcard, filter
                    filters[0] = analysisOptions.getSearchType();
                    filters[1] = analysisOptions.getSearchRegion();
                    filters[2] = analysisOptions.getSearchFilter();
                    filters[3] = analysisOptions.getSearchChildren();
                    filters[4] = analysisOptions.getSearchWildcard();
//                    filters[0] = parsedArguments[2];
//                    filters[1] = parsedArguments[3];
//                    filters[2] = parsedArguments[6];
//                    filters[3] = parsedArguments[4];
//                    filters[4] = parsedArguments[5];

```

Now the `returnArguments()` method can be removed. Do NOT comment it out because it clutters your code that is finally becoming readable.
If you want it back (or have a look at it), simply go to a previous commit.

### Remove unneccesary comments  

Good code reads like a book! If you need it for yourself, your code is probably not very good.
API methods should be very well documented of course, but inline comments or private methods comments should ideally not be necessary. Keep them to a minimum! They get outdated and are becoming damaging by themselves. A comment like `/*builds the options*/` is entirely unnecessary!

### The result

This is the `CLIParser` class after complete refactoring:

```java
package nl.bioinf.gff_query.io;
import org.apache.commons.cli.*;

public class CLIParser {
    public static final String OPTION_HELP = "help";
    public static final String OPTION_INFILE = "infile";
    public static final String OPTION_SUMMARY = "summary";
    public static final String OPTION_FETCH_TYPE = "fetch_type";
    public static final String OPTION_FETCH_REGION = "fetch_region";
    public static final String OPTION_FETCH_CHILDREN = "fetch_children";
    public static final String OPTION_FIND_WILDCARD = "find_wildcard";
    public static final String OPTION_FILTER = "filter";
    /*private -> testing*/ Options options;
    /*private -> testing*/ CommandLine commandLine;
    private DefaultParser parser;
    private HelpFormatter formatter;
    private Options helpOptions;

    public CLIParser() {
        this.parser = new DefaultParser();
        buildOptions();
    }

    private void buildOptions() {
        this.options = new Options();
        this.helpOptions = new Options();
        Option helpOption = Option.builder("h")
                .longOpt(OPTION_HELP)
                .required(false)
                .hasArg(false)
                .desc("Gives usage instructions")
                .build();
        options.addOption(helpOption);
        helpOptions.addOption(helpOption);
        Option infileOption = Option.builder("i")
                .longOpt(OPTION_INFILE)
                .required(true)
                .hasArg(true)
                .desc("The input gff3 file, as absolute or relative path")
                .build();
        options.addOption(infileOption);
        Option summaryOption = Option.builder("s")
                .longOpt(OPTION_SUMMARY)
                .required(false)
                .hasArg(false)
                .desc("lists a summary of the gff3 file")
                .build();
        options.addOption(summaryOption);
        Option fetchtypeOption = Option.builder("ft")
                .longOpt(OPTION_FETCH_TYPE)
                .required(false)
                .hasArg(true)
                .desc("Lists all features of the requested type, e.g. \"CDS\"")
                .build();
        options.addOption(fetchtypeOption);
        Option fetchregionOption = Option.builder("fr")
                .longOpt(OPTION_FETCH_REGION)
                .required(false)
                .hasArg(true)
                .desc("Lists all the features that reside completely within the given region; specified as start..end, e.g. \"250000..260000\"")
                .build();
        options.addOption(fetchregionOption);
        Option fetchchildrenOption = Option.builder("fc")
                .longOpt(OPTION_FETCH_CHILDREN)
                .required(false)
                .hasArg(true)
                .desc("Lists all child features of the given feature ID, e.g. \"PGSC0003DMT400039136\"")
                .build();
        options.addOption(fetchchildrenOption);
        Option findwildcardOption = Option.builder("fw")
                .longOpt(OPTION_FIND_WILDCARD)
                .required(false)
                .hasArg(true)
                .desc("Lists all features for which the \"name\" attribute matches the given wildcard, specified as regex pattern, " +
                        "e.g. \"[dD]efesin\"")
                .build();
        options.addOption(findwildcardOption);
        Option filterOption = Option.builder("f")
                .longOpt(OPTION_FILTER)
                .required(false)
                .hasArg(true)
                .desc("Lists all features that pass all of the given filters specified in the format string defined as" +
                        ": \"source|score|orientation|maximum_length|minimum_length\" where the filters should be relevant to the " +
                        "given attribute. Suppression of an individual filter is indicated using an asterisk (*)." +
                        "ORIENTATION should be defined using a \"+\", \"-\" or \".\" character")
                .build();
        options.addOption(filterOption);
    }

    public void printHelp() {
        this.formatter = new HelpFormatter();
        String footer = "\nPaths are platform independent.";
        this.formatter.printHelp("GffQuery", "Version: 1.1-SNAPSHOT", this.options, footer, true);
    }

    public boolean isHelpRequested(String[] args) {
        try {
            CommandLine commandLine = this.parser.parse(this.options, args);
            return (commandLine.hasOption(OPTION_HELP));
        } catch (ParseException e) {
            return true;
        }
    }

    public void parseCommandLineArguments(String[] args) throws ParseException {
        this.commandLine = this.parser.parse(this.options, args);
    }

    public GffAnalysisOptions getAnalysisOptions() {
        GffAnalysisOptions gffAnalysisOptions = new GffAnalysisOptions();
        if (this.commandLine.hasOption(OPTION_INFILE)) {
            gffAnalysisOptions.setInFile(this.commandLine.getOptionValue(OPTION_INFILE));
        }
        if (this.commandLine.hasOption(OPTION_SUMMARY)) {
            gffAnalysisOptions.setSummary(true);
        }
        if (this.commandLine.hasOption(OPTION_FETCH_TYPE)) {
            gffAnalysisOptions.setSearchType(this.commandLine.getOptionValue(OPTION_FETCH_TYPE));
        }
        if (this.commandLine.hasOption(OPTION_FETCH_REGION)) {
            gffAnalysisOptions.setSearchRegion(this.commandLine.getOptionValue(OPTION_FETCH_REGION));
        }
        if (this.commandLine.hasOption(OPTION_FETCH_CHILDREN)) {
            gffAnalysisOptions.setSearchChildren(this.commandLine.getOptionValue(OPTION_FETCH_CHILDREN));
        }
        if (this.commandLine.hasOption(OPTION_FIND_WILDCARD)) {
            gffAnalysisOptions.setSearchWildcard(this.commandLine.getOptionValue(OPTION_FIND_WILDCARD));
        }
        if (this.commandLine.hasOption(OPTION_FILTER)) {
            gffAnalysisOptions.setSearchFilterAsString(this.commandLine.getOptionValue(OPTION_FILTER));
        }
        return gffAnalysisOptions;
    }
}
```

## Take over

I challenge you to now take class `GffQuery` and make it look good! Check out tag 0.2 of branch `master` (commit 19a08f4).


