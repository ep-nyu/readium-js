EPUBCFI
=======

# The EPUB 3.0 CFI library

This library provides support, in javascript, for parsing and interpreting EPUB 3.0 CFIs. jQuery is the only dependency.

# How to use the CFI library

Note: Steps 3. and 4. will change soon. The requirement to retrieve a package document, explicitly parse a CFI and pass the result to
the Interpreter, will be removed. The intention is to create an API that only requries the user to specify the URL of an EPUB's package document and the elements to inject for a particular type of CFI.

1. Get a copy of the library. Currently, a development version of the (library)[https://github.com/justinHume/EPUBCFI/blob/master/epub_cfi.js] is available in the Github repository. When the library becomes more stable, a minified version will also be made available as a separate download. 

2. Add the CFI library to your code using a script tag, and make sure you have included jQuery:

    <script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js"></script>
    <script src="epub_cfi.js"></script>

3. Set the package document URL and retrieve a package document object:

    EPUBcfi.Config.packageDocumentURL = "http://something/something";
    EPUBcfi.Config.cfiMarkerElements.textPointMarker = '<span class="cfi_marker"></span>';

    $packageDocument = do_some_stuff_to_get_a_jQueryed_package_document_object;

4. Parse a CFI and inject an element for that CFI:

    cfi = 'epubcfi(/6/18!/4/2/4:2)';

    try {
    
        ast = EPUBcfi.Parser.parse(cfi);
        $resultWithInjection = EPUBcfi.Interpreter.injectCFIReferenceElements(ast, $packageDocument, pathToPackageDoc);
    }
    catch (err) {

        // Do something with the error
    }

The result of this will be to inject the '<span class="cfi_marker"></span>' HTML element into a position in the EPUB pointed to by the CFI.

# Setting up the development environment

There are a number of dependencies for this project: 

* Ruby
* Rake
* The ERB template gem
* Git
* (jasmine)[http://pivotal.github.com/jasmine/]
* (PEG.js)[http://pegjs.majda.cz/]. PEG.js is both a library and a command-line tool. The project assumes that the PEG.js command line tool can be found on your path. You can change this in the Rake file. 

Once all that is good to go, clone the (Github)[https://github.com/justinHume/EPUBCFI] repository. 

There are a number of Rake tasks to help you with development: 

* Generate a parser from the .pegjs grammar: rake gen_parser
* Run the CFI library unit tests: rake jasmine
* Generate a parser AND run the tests: rake test_parser
* Generate a vesion of the stand-alone library: rake gen_cfi_library

That last Rake task will generate a single javascript file that contains all of the library components. For development purposes, the different library components are in separate files in the development environment.

# Future development priorities

This is a very early version of this library and there are currently no guarantees about what will change in the next little while. This library will evolve as I develop a better understanding of use cases for CFIs. 

The following are the development priorities (in order), going forward:

* Add support for asynchronous resource retrieval.
* Add CFI ID and text assertions.
* Add escape characters to the grammar.
* Support the Georgia sample in Readium; it contains a CFI-based table of contents.
* Add the CFI text-range terminus.
* Add iframe indirection.
* Add additional terminus types.
* Decide on some guarantees about the library API.

# How it works, for developers

For the purposes of this library, CFIs can be viewed as a Domain Specific Language (DSL) for creating unique, recoverable and sortable indexes for EPUBs. Most of what I know of DSLs I learned from reading (Martin Fowler's work)[http://martinfowler.com/bliki/DomainSpecificLanguage.html], so I'll just link him in here and let him describe DSL concepts, if you haven't been exposed to them.

This CFI library is implemented using a number of standard patterns for Domain Specific Languages (DSLs). Before getting into the rationale for using DSL patterns, I'll go over the requirements for the CFI library.

* Fully implement the EPUB 3.0 CFI specification.
* Provide a stand-alone javascript library that enables a reading system to do useful things with a CFI. 
* Demonstrate the implementation of the CFI specification and library; that is, make the methodology, code and development environment accessible to reading system designers and users of the EPUB standard.

Given these requirements, the rationale for the DSL approach is two-fold. First, to do something useful with a CFI, it must essentially be parsed and interpreted. This part of the problem is irreducible. Even if a non-language-tool approach were taken, the eventual result would almost certainly be to write something that resembles a lexer-parser-interperter-something. Given that the EPUB CFI grammar is fully specified and that DSL tools and methodologies are mature and available, it makes sense to leverage existing DSL approaches for this type of work. Using well-understood and practiced patterns leads to a (hopefully) clean, robust and extensible solution. This is especially true when considering benefits like built-in syntax error handling that comes free-ish with DSL tools like parser generators. 

Second, using widely available language-tool patterns aids in making the CFI library more accessible to developers and implementors. I think this is the case for all the standard reasons that design patterns, whether loosely or rigorously applied, are beneficial. Primarily, it is easier to understand - or learn about - a common pattern, as opposed to whatever amorphous collection of objects I might have invented on my own. 

## The CFI library architecture

The CFI library consists of a number of components, with reponsibilities as follows:

* Configuration: Maintains properties important to the behaviour of the CFI library.
* Parser: Lexes and parses (in one step) a CFI string. It produces either syntax errors or an Abstract Syntax Tree (AST) representation of the CFI, in JSON format.
* Interpreter: Walks the JSON AST and executes instructions for each node in the AST. The result of executing the interpreter is to inject an element, or set of elements, into an EPUB content document. These element(s) will represent a position or area in the EPUB referenced by a CFI.
* Instructions: The implementation of a single statement in the CFI language.
* Runtime errors: A set of errors that might be thrown as a CFI AST is interpreted. 

The following describes the rationale for each component:

## Lexing and parsing

The PEG.js parser generator is used to generate a lexer/parser from a Parsing Expression Grammar (PEG) that represents the CFI language.Given that an Extended Backus-Naur Form (EBNF) representation of the CFI grammar is provided by the EPUB spec, using a parser generator as the mechanism for building and maintaining a lexer/parser seems appropriate for a number of reasons. 

First, using a parser-generator with a well-specified grammar ensures a more complete (built-in syntax error generation, etc.) and error-free lexing/parsing solution, as opposed to writing the lexer/parser by hand. 

Second, it is easier to maintain a parser generator solution over time, as changes to the spec or grammar can be more easily versioned, accommodated and understood if represented by a nice pretty grammar, rather than directly in code. 

Third, using a parser generator enables a separation-of-concerns between lexing/parsing and whatever comes after (interpretation, in this case). This is _good_ because separation-of-concerns makes the life a developer much, much better. 

Finally, in regards to the rationale for using a Parsing Expression Grammar (PEG): The PEG.js library seems to provide a simple generator, with a javascript target, that has decent how-tos and other documentation. Beyond that, there is nothing that I can see in the CFI language that necessitates, or is limited by, the use of a PEG as opposed to a generator based on an EBNF, like ANTLR, or something similar. 

tl;dr, I chose a generator based on a PEG because PEG.js seemed easy and useful, rather than for some deep language reason. 

## Interpreter and instructions

The interpreter is responsible for walking the AST and calling instructions that do something to interpret a CFI. This _something_ is, at the moment, to inject some arbitrary user-specified HTML at the location pointed to by the CFI. 

The rationale for the interpreter and the instruction components being separate is the standard set of engineering reasons: Separation-of-concerns, extensibility, better abstraction etc. 

To illustrate, while there is currently a one-to-one correspondence (with one exception) between an interpreter function and an instruction function, it is likely that the in the future the behaviour of the interpreter will become more complex. It may act on a set of CFIs, load resources differently, etc. These types of changes would have no impact on the instructions being executed by the interpreter, hence the separation. 

## Error handling

The interpreter and instructions can generate a variety of runtime interpretation errors. Right now the intention is to provide some basic information for CFI interpretation errors (syntax errors are generated by the parser). There is not much to say about this other than that this component will likely evolve with a better understanding of the type of runtime errors a CFI might generate and the use cases for the this library. 


