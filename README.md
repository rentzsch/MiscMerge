Release 2.0.6  Copyright 2000 by Don Yacktman and Carl Lindberg.  All Rights Reserved.

# The MiscMergeKit

*This README is directly converted from [MiscMerge.rtf](MiscMerge.rtf), reformatted as Markdown for easier access.*

## Introduction


The MiscMergeKit implements a generic engine which may be used to combine data records and template files.  A template file is defined using a *merge language* (described below) and contains *blanks*.  The blanks are filled in with data taken from a data record, which is any object.  The name of the blank is used to make a `-valueForKey:` call on the data record.  Other merge commands allow conditional text, comments, and other features.  The engine itself is meant to be embedded in a program and is flexible to be used for many purposes:

* Document merge capability (inside word processors or other user apps). 
* Programmatic generation of code in C, Objective C, or other languages. 
* Programmatic generation of documentation. 
* Programmatic generation of Web pages. 
* Anything else you can imagine… 

## MiscMergeKit Classes and Protocols

### Basic Merging 

The MiscMergeEngine class is the starting point for a basic, one-shot merge.  All you have to do is:

1.  Initialize a `MiscMergeTemplate` instance for the template you wish to use
2.  Get the data record object
3.  Pass the `MiscMergeTemplate` and data object to the `MiscMergeEngine`
4.  Start the merge
5.  Do something with the results, an `NSString` instance, that the `MiscMergeEngine` returns to you.

If there is more than one data record, then a `MiscMergeDriver` instance should be used to perform the merge.  You pass it an initialized `MiscMergeTemplate` and an `NSArray` filled with a data object for each merge.  You get back a new `NSArray` filled with `NSString`s, the results of the individual merges.  The `NSString`s are in the same order as the objects on the input side, so it is easy to correlate the output to the input.

### Advanced Features

If you need to actually modify the merge language, it is possible to do so.  The API to the `MiscMergeCommand` class is provided to allow you to add commands or override existing commands.  Since all new commands are subclasses of `MiscMergeCommand`, there is a rich set of functions available to aid in parsing and writing the behavior of new commands via inheritance.  There are methods available to aid in creating commands that use an `if`/`else`/`endif` semantic or some other type of conditional.  The `MiscMergeDriver` informal protocol is provided to aid in creating custom `MiscMergeDriver` style classes if the provided class is inadequate for any reason.  The current driver is flexible with choice of engine, so in most applications it works well.  If you are interested in any of these features, you should consult the applicable class documentation, which describes the details of these features.  Be sure to read the class documentation since there are many more options and hooks to allow customization of the engine to fit your needs.  Become especially familiar with some of the esoteric `MiscMergeEngine` methods.  Each command is sent the id of the `MiscMergeTemplate` when parsing and the id of the `MiscMergeEngine` that invoked it when executing so that callbacks to the engine (or template) are possible.  Many of the built-in commands use these callbacks to detect and alter the engine's state.

## MiscMergeKit Merge Language

The merge language is quite simple.  Two delimiters are used to tell the template parser when a merge command begins and ends.  By default, commands are expected to be enclosed in pairs of `«` and `»`, but you can change this to brackets (`{` and `}`), double arrows (`<<` and `>>`), or any other pair of arbitrary-length strings.  The same string can be used as the opening and closing delimiter, though this defeats the delayed-parse feature (not necessarily a bad thing).  The first word after the opening delimiter is used as the command name.  It is used to look up a command class.  If the class is not found, then the word is taken to be a key of the data object and the merge command will substitute the value returned by the object for the merge command on the output.  So the simplest example would be:

Template:

    This is a sample template for {name}.

Dictionary:

    name = "Don Yacktman"

Since there is no `name` command, the value of the key `name` will be substituted into the output, to give the following output:

    This is a sample template for Don Yacktman.

When searching for keys, the `MiscMergeEngine` can be configured to resolve the values as far as it can, though this is off by default since it can cause some unexpected problems.  So, for example, one value in the object could be another object in the dictionary--causing an indirection to take place.  If keys aren't found in the data object, an engine-specific dictionary is consulted, and then a global dictionary.  If still not found, then the key is returned as a literal value to be inserted into the template.  Here are some examples:

Template:

    This is a sample template for <#name#>.

Dictionary:

    <empty>

Output:

    This is a sample template for name.

Dictionary:

    name = "fullName"
    fullName = "Don Yacktman"

Output:

    This is a sample template for Don Yacktman.

If an actual command is found, then that command will be executed.  What does or doesn't get placed into the output of the merge depends upon the command used.  In fact, the syntax of the command itself depends upon the command.  The command descriptions below detail what parameters (if any) are expected to follow a particular command.  Obviously, any commands you create will have the syntax you specify.  If a command contains nested merge commands, then it will not be parsed until merge time.  In fact, a special `MiscMergeEngine` will be created to merge the command and then it will be parsed.  Although this considerably slows performance, it allows the merge commands to change depending upon the input data!  This ability turns out to be sometimes useful…but it can be dangerous if you aren't careful.  (I have used this feature primarily with custom commands I have created, where the parameters come from values in the data object for the merge.)

Finally, note that the command keywords are case insensitive.  Thus, `Copy`, `copy`, and `COPY` all refer to the same command as far as the `MiscMergeTemplate` parsing machinery is concerned.  For those who remember, many of the commands are similar to the merging commands used by the WriteNow.app distributed with NEXTSTEP 2.1 and earlier since that language was used as an example when this language was created.  This language is both richer and extensible, however.

## MiscMergeKit Expressions

The MiscMergeKit allows complex expressions, just like C or Java for various arguments to its commands.  Full precedence rules are in effect.  This list describes the precedence order of commands for expressions.  Precedence is evaluated top to bottom, and left to right.

    () key
    - !
    * / %
    + -
    <= =< le >= => ge < lt > gt
    <> >< != neq ne == = eq
    && and
    || or


### Key Value 

The key value above is a string that is normally interpreted by the `MiscMergeEngine` to determine if there is some value that has been stored under the name. Otherwise the key name is taken as the value itself.  However, there are two options for the key itself.  You can single-quote a string, and it will prevent the `MiscMergeEngine` from attempting to evaluate the string for a value, it will just be treated as a literal.  You can also double-quote a string.  Double quoted strings will be evaluated.  Typically, you would use a double quoted string to be able to access a variable with a space in the name. Normally, if key does not evaluate to an object, a blank string will be returned.  However, with the `option nilLookupResult` key command, the return value can be made to be the key string itself.

### Primary Expressions

Primary expressions are the only kind of expressions that can be specified for a number of arguments to commands.  Primary expressions are unique in that they can be uniquely identified where they start and end

## MiscMergeKit Commands

Following is a list of all the MiscMerge commands that come with the kit by default.  Each command name is listed in bold, followed by a list of the arguments that and follow it.  The syntax for the arguments is as follows:

* `str`	A string is a series of alphanumeric characters with no spaces.
* `text`	A string is a series of alphanumeric characters.
* `key`	A key is a string that is used to either store or retrieve a value from the merge context dictionary.
* `exp`	This is represented by a full expression, as defined in the expression section above.
* `pexp`	This is represented by a primary expression, as defined in the expression section above.
* `name:type`	The name portion just represents a descriptive name for what the argument is and type will be one of the five argument types above which defines what the argument should be.
* `[ arg ... ]`	Defines an optional argument, and `...` means the previous argument can be repeated.

### **break**

Immediately exits a block command, ie: `foreach`, `loop`, `while`.

### **continue**

Jumps to the end of a block command, ie: `foreach`, `loop`, `while`. The conditional code for that block will be then executed and it will proceed as normal.

### **copy text**

Copies all the text following the `copy` keyword to the merge output.  Plain text in the template is turned into `copy` commands internally by the `MiscMergeTemplate` parsing mechanism.  You'll probably never use this as a command—but should remember to avoid using the word `copy` as a key in the merge dictionary, since that would invoke this command!

### **comment text**

This command is a no-op.  Anything following the `comment' keyword is ignored and discarded.  Nothing is inserted in the merge output.

### **date**

Places the current date in the merge output.  It takes one optional parameter, an `NSCalendarDate calendarFormat` which is used to format the date.  By default, this is of the form `Month day, year` (i.e. `"%B %d, %Y"`) which will format as `July 21, 1995`.

### **debug text**

Copies all the text following the `debug` keyword to the standard error output.  This can be used to try to debug a template file.

### **field exp**

The text following the keyword `field'` is parsed and processed as a full expression, evaluated, and the results of the expression are placed in the merge output.  This is the long form and can be used to get around problems with keys that have the same names as commands.  For example, `$<copy>$` won't work for retrieving the value for `copy` from the data object, but `$<field copy>$` will work.  

### **foreach item:key array:key [ label:str ]**
### **endforeach [ label:str ]**

These commands implement a loop.  The `foreach` command requires three parameters.  The first parameter is a variable name to be used as an iterator.  Each time through the loop, this variable will take on a new value.  A variable using the same name but with `Index` appended is also available in the context of the loop, and will contain the current loop iteration number (starting with 0).  When the second item evaluates as a `NSDictionary`, an additional name with with `Key` appended is also available in the context of the loop, and will contain the key that the item value was found at.  The second parameter is the name of a variable which contains an instance of the `NSArray` or `NSDictionary` class (or one of their subclasses).  The final parameter is a special tag.  The `endforeach` command can take a single parameter, again a tag.  The tags for each set of matching `foreach`/`endforeach` commands are expected to match.  If they do not, then the template is assumed to be incorrectly constructed and the merge is aborted with an error message.  If the array is empty, then no iterations are performed and nothing appears in the output.

Template:

    <tr>[foreach value theRow row1]<td>[value],[valueIndex]</td>[endforeach row1]</tr>

Dictionary:

    theRow = ("5", "10", "20", "30")

Output:

    <tr><td>5,0</td><td>10,1</td><td>20,2</td><td>30,3</td></tr>

### **if exp**
### **elseif exp**
### **else**
### **endif**

These four commands allow conditional text output with a merge.  Here would be a way to print out a different string of text based upon the value of the key `salary`:

Template:

    Congratulations!  You qualify for our offer for a free Visa [$if salary > 35000$]Gold[$else$]Classic[$endif$] card!

Dictionary:

    salary = "20000"

Output:

    Congratulations!  You qualify for our offer for a free Visa Classic card!

Dictionary:

    salary = "40000"

Output:

    Congratulations!  You qualify for our offer for a free Visa Gold card!

The expression is evaluated to see if it is true.  A true value is defined by a non-zero number, a non-zero length string, or a non-null object. 

### **setlocal key = exp**
### **setmerge key = exp**
### **setengine key = exp**
### **[set | setglobal] key = exp**

This allows a value for a key to be determined.  For example, `<<identify name = f0>>` will make the key `name` return `f0`.  Depending on how the merge engine is configured, the key `f0` may then be searched for.  If not found, the text `f0` would be returned, otherwise the value of the key `f0` would be returned.  This allows aliases for key names to be created as well as simple setting the values of keys.  Note that the statement requires an `=` operator for it to be parsed correctly.

The three commands operate on different context dictionaries and allow for differing levels of locality of reference.  The `set` command saves it's value into the global dictionary which is used for all merges. This dictionary will be searched last to determine a value. The `setmerge` command saves it's value into a merge's dictionary. Values at this level will only be retained until the end of the current merge. This dictionary will be searched before the data object, so settings done in this manner can mask values in the main data object.  The `setlocal` command saves it's value into the first context dictionary that is on the dictionary stack. In normal use, this operates the same as the `setmerge` command.  However, when a procedure is invoked via the call command, a new dictionary is placed on the context stack.  Using the `setlocal` command from within a procedure allows for that procedure to have truly local variables that do not affect the rest of the merge.

### **include filename:str [ startdelim:str enddelim:str ]**

This allows an external template file to be inlined into the current template.  It takes one parameter, the path to the template to be loaded, and two optional arguments, strings that define the start or end delimiter.  Internally, a separate `MiscMergeTemplate` instance is used to load and parse the template.  The included file is expected to have the same delimiters by default, but you can specify what delimiters it should use.  The path can be an absolute path or relative to the current directory.  The `MiscMergeTemplate` class allows a delegate to be set to provide other processing to find the file.  Typically this would be to define a standard file location to find files.

### **index array:key index:pexp**

If one of the dictionary values is actually an instance of `NSArray` or one of its subclasses, then this command can be used to access a single value from the array.  Two parameters are required.  The first is the variable name and the second is the index (starting with zero) of the string to use.  As an example:

Template:

    Please hand me that $$index theList 1$$.

Dictionary:

    theList = ("apple", "bananna", "orange")

Output:

    Please hand me that bananna.

### **loop item:key start:pexp end:pexp increment:pexp [ label:str ]**
### **endloop [ label:str ]**

These commands implement a loop.  The `loop` command requires four parameters and an optional fifth.  The first parameter is a variable name to be used as an iterator.  Each time through the loop, this variable will take on a new value.  The second parameter is a start value, while the third parameter is an end value.  The fourth parameter is a step value.  The second, third, and fourth parameters are all integers; no floating point math is supported.  The final parameter is a special tag.  The `endloop` command takes a single parameter, again a tag.  The tags for each set of matching `loop`/`endloop` commands are expected to match.  If they do not, then the template is assumed to be incorrectly constructed and the merge is aborted with an error message.

Template:

    He ate {loop value 10 50 10 loop1}{value} {endloop loop1 }times.

Dictionary:

    <empty>

Output:

    He ate 10 20 30 40 50 times.

### **procedure name:str [ var:str ... ] [ var?:str ... ] [ var...:str ]**
### **endprocedure**
### **call name:str [ pexp [ pexp ... ]]**

This set of commands allows an internal "procedure" to be defined inside the template directly, and then called with different arguments.  The `procedure` command has one required parameter, the procedure name.  Any other parameters are names of variables expected to be passed to the procedure, and can be used as keys in the context of the procedure definition.  An `endprocedure` command (no parameters) ends the procedure definition.  A procedure can call itself recursively.

A `call` command is used to call a previously-defined procedure.  It takes one mandatory argument, the procedure name to call, plus any number of arguments to pass to the procedure.  These arguments can be key names that will be resolved via the data object or raw values.  Naturally, the number of arguments in the call command should match up with the number of arguments the procedure is expecting.

The variable names in the procedure command have some options as far as naming and affect what happens when the call command is used.  Any variable name with a `?` at the end of the name will considered an optional argument.  Optional arguments must come after non-optional ones.  If a procedure is executed but no value exists in the spot for a optional command, it will be assigned a blank string value.  The `?` will not be used as part of the key name of which the value is saved under.  One other variable can be listed in the procedure's argument list and that has a `...` at the end of the variable name.  This defines a variable argument list and must be at the very end of the procedure command.  Any arguments in a call command that start at that arguments position and afterwards are added to an array that is saved to the variable name, without the `...` extension.

### **next**

If a MiscMergeDriver initiated the merge, then the `next` command will cause the next data object to be loaded.  This allows merges for multiple records to be placed into a single output merge.  Note that the `MiscMergeDriver` will add in empty output strings as placeholders for the output `NSArray`, since the merge for the next record will not be performed.  This command might be used, for example, in creating pages of address labels.  You could put several labels on one page this way.

### **omit**

Causes the merge to be aborted.  The output string will be empty.  This command can be used in an `if` statement to conditionally abort merges depending upon values of keys in the merge dictionary.

## For Advanced Users - The option Command

Their is one final command, option, that can be used in a template file. This command is used to alter the behavior of the MiscMerge processing and is generally only recommended for advanced users.  The option command is followed by one of the following arguments:

### **delimiters**

This option is followed by two parameters, which specify a starting and ending delimiter. The rest of the template will be processed using these delimiters. Be aware that the command itself must end with the delimiter that was in use before this command was called.  For instance if `{` and `}` are the starting and stopping delimiters, ie:

	{option delimiters ( )}

### **betweenWhitespace**

This option is used to determine how MiscMerge processes whitespace that surrounds any text between commands. It is most useful to be able to create procedures with nice indenting without ending up with lots of excess whitespace. This option takes one of four keywords.

#### keep

This option is just the default mechanism of MiscMerge. No processing happens.

#### trim

Any whitespace at the beginning or end of a block of whitespace is trimmed off.

Example Template:

    '{foreach item array do}
        {if itemIndex gt 0}
            ,
        {endif}
        {item}
    {endforeach do}'

Output for values `(doug, jon, carl)`:

    'doug,jon,carl'

As can be seen, the template is easy to read versus the alternative:

Non Trim Template:

    '{foreach item array do}{
        if itemIndex gt 0},{
        endif}{item}{
    endforeach do}'

#### keepNonBlank

Only text that includes non-whitespace text will be output.

Example Template:

    '{foreach item array do}
        {if itemIndex gt 0} , {endif}
        {item}
    {endforeach do}'

Output for values `(doug, jon, carl)`:

    'doug , jon , carl'

#### ignoreCommandSpaces

This option was designed to be able to have templates that output C code. It will remove any whitespace plus a newline that follows the ending delimiter of a command and remove whitespace before the start of a command up to but not including the last newline.

Example Template:

    {foreach item array do}
        int {item};
    {endforeach do}

    {foreach item array do}
        if ( {item} == 0 )
            NSLog(@"{item} must be not equal to zero");

    {endforeach do}
    ...

Output for values `(doug, jon, carl)`:

	int arg1;
	int arg2;

	if ( arg1 == 0 )
		NSLog(@"arg1 must be not equal to zero");

	if ( arg2 == 0 )
		NSLog(@"arg2 must be not equal to zero");

    ...

Note how while there is one newline before the if statement, no newline is output, but at the end of `NSLog` there is two newlines and they are both output.

### **failedLookupResult**

This option is used to determine how MiscMerge handles a failure to lookup a key path.  MiscMerge will attempt to determine if the first part of a key path exists in one of the contexts that it resolves.  If the key path is a complex multi-part path, the first portion is still the only portion that is evaluated. Consider:

Template:

    {user} {user.name}

Dictionary:

    <empty>

Output:

    user user.name

Dictionary:

    user = {};

Output:

    '{} '

Dictionary:

    user = nil; <- This must come from a Objective-C method

Output:

    ' '

Normally, if the first part of the path is found, whatever the full key path returns is returned in the output. If the first part of the key isn't found, the full key path is returned.  Note in the second and third examples, only `user` exists, while `user.name` does not.  MiscMerge was designed with the expectation that the user would provide all the correct data to the engine.

However, in the case of procedure calls, this can cause some unexpected behavior.  For example:

Template:

    {procedure printorblank item}{if item ne ''}{item}{else}isBlank{endif}{endprocedure}
    '{call printorblank user} {call printorblank user.name}'

Dictionary:

    <empty>

Output:

    'user isBlank'

This may be what you want, but you may really want to know that 'user; does or doesn't actually exist. This option allows this behavior to be changed.  The following keywords will affect what happens when MiscMerge does not find the first part of a key.

#### key

This is the default behavior in that the full key path is returned if the initial part of the key path is not found.

#### keyWithDelims

This is the same as the previous except it returns the key path with the current delimiters surrounding it.

#### nil

Return nil if the initial part of the key path is not found.

#### keyIfNumeric

Return the key path if the initial part of the key path is not found if it is a numeric value, or nil otherwise. This allows you to use numbers in conditionals without having to enclose them in single quotes to ensure that they are treated as literals.

### **nilLookupResult**

This option is similar to the `failedLookupResult` option, but controls what happens when the initial key path is found, but the evaluation of that key path returns nil. The following keywords will affect what happens when this happens.

#### nil

This is the default behavior in that nil is just returned.

#### keyIfQuoted

If the key path resulted in nil, but the key was double quoted, then return the key path.  This allows you to have key path lookups, but default to a key path if it did not evaluate to anything. This can be handy for debugging purposes or places where you might want a default value for some output but not for other lookups.

#### key

This just returns the key path if the value was nil.

#### keyWithDelims

This is the same as the previous except it returns the key path with the current delimiters surrounding it.

### **recursiveLookups**

This allows a template designer to turn on recursive lookups in a template.  Because recursive lookups are dangerous in that they can end up wandering around to places you might not expect, this allows a template designer to turn it on at individual places. This option takes one of three parameters: `yes`, `no`, or a number. The parameter `yes` will tell MiscMerge to attempt a recursive lookup until it hits a recursion limit which is set at 100 by default. It is presumed that if a recursive lookup happens more than 100 times (and really, it is doubtful a recursion past 10 or so times is valid), then the something is  wrong in the template.  The last value that it found will be returned.  The template designer can also specify a number which is then used to turn recursion on and specify what the recursion limit is.

*All trademarks used herin are owned by their respective owners.  We're just borrowing them to give you a frame of reference by which you can understand what the MiscKit does.*

