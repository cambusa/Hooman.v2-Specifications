# Hooman metalanguage specifications - Version 2

__Creator: Rodolfo Calzetti__

__License: Creative Commons Zero v1.0 Universal__

Hooman is a metalanguage that allows you to represent data in a hierarchical way as in XML, JSON and YAML.

Hooman is deliberately devoid of any syntactic tinsel and the hierarchy is expressed through indentations as in Python: 
this makes the language highly human-writable. 
You don't have to use "equal" for the assignment; values don't have to be enclosed in quotation marks; 
you don't have to open and close brackets or tags; blank lines are ignored. 

## Variables

A **variable** identifier is the first word from the left in a line - consisting of English letters, 
digits and underscores - which occurs after spaces or tabs.

The language is case insensitive.

By default, a tabulator equals four spaces, but you can use a different number of spaces for indentation: 
what's important is that the choice be consistent throughout the whole document. 
The first indentation is taken to establish equivalence with a tabulation character; 
if the first indentation were obtained by means of a tabulator then the equivalence would remain four spaces per tabulator.

## Values

A **value** assigned to a variable can be of two types:

* a **simple** value, consisting of the string following a variable on the same line (with leading and/or trailing spaces and tabs removed).

* a **complex** value, consisting of the sub-document underlying the variable and indented by a tabulation.

Simple values are just strings, but we will see that it is possible to define custom types using syntactic rules.

If you want to assign a multi-line string to a variable, we can use the guillemot notation (```<<``` ... ```>>```).

Let's take a simple example of a hooman document. 

```
hooman 
    version 1

company
    employees
        1
            name Johann Sebastian Bach
            info
                <<
                    Bach was born in 1685 in Eisenach, 
                    in the duchy of Saxe-Eisenach, into an 
                    extensive musical family.
                >> 
        2
            name Emmy Noether
            info
                <<
                    Noether was born to a Jewish family 
                    in the Franconian town of Erlangen; her father was 
                    the mathematician Max Noether.
                >>
```

Note that the recommended style for variables is to use only lowercase characters to avoid unnecessary key presses.

In this example, ```company```, ```employees```, ```1``` and ```2``` are complex variables, 
whereas ```name``` and ```info``` are simple variables.

You can tag the guillemots in order to inform about the format of the underlying string.

For example, you could write:

```
chapter 
    <<latex
        $\int{\frac{Mx+N}{p(x)}dx}=\lambda \times ln(p)+\mu 
        \times arctg\frac{p'}{\sqrt{-\Delta}}$
    >>
```

## Remarks

A line starting with three asterisks ```***``` will be ignored by the parser: this will allow you to comment on parts of the document with explanatory notes:

```
chapter 
    *** Method for integrating fractional rational functions with negative delta
    <<latex
        $\int{\frac{Mx+N}{p(x)}dx}=\lambda \times ln(p)+\mu 
        \times arctg\frac{p'}{\sqrt{-\Delta}}$
    >>
```

## Inclusions

A subdocument assigned to a variable can be included from another file using the back arrow notation ```<--```.

For example, let the following files be given: 

*c:\hoomans\main.fud*

```
hooman
    version 1
    <-- c:\hoomans\syntax.fud

company
    employees
```

*c:\hoomans\syntax.fud*

```
syntax
    structure
        company
            employees

    rules
```

The second file will be incorporated into the first obtaining the equivalent of the following document: 

```
hooman
    version 1
    syntax
        structure
            company
                employees

        rules

company
    employees
```

Inclusion can also be used to include text; for example we can write: 

*c:\hoomans\book.fud*

```
hooman
    version 1

chapter1
    <<
    <-- c:\hoomans\chapter1.fud 
    >>
```

*c:\hoomans\chapter1.fud*

```
People assume that time is a strict progression 
of cause to effect,  but actually, from a nonlinear, 
non-subjective viewpoint, it's more like a big ball 
of wibbly-wobbly, timey-wimey... stuff.
```

Obtaining the equivalent of: 

```
hooman
    version 1

chapter1
    <<
        People assume that time is a strict progression 
        of cause to effect,  but actually, from a nonlinear, 
        non-subjective viewpoint, it's more like a big ball 
        of wibbly-wobbly, timey-wimey... stuff.
    >>
```

Note that if we had written:

```
hooman
    version 1

chapter1
    <<
        <-- c:\hoomans\chapter1.fud 
    >>
```

we would not have gotten any inclusion, as the parser would have interpreted ```<--``` as
part of the text: the text itself does not need any escaping of characters inside it. 

### Virtual inclusions

When the parser encounters the arrow clause ```<--```, an event is triggered which
allows the developer to intercept, block and overwrite the inclusion of the file:
the parameter passed as a file path can be reinterpreted as a key
to propose some text to include. Typically, the text could be read from a database. 

## Redefinition of variables

It can be useful to override the value of both simple and complex variables.

For example, we could include a file of defaults and then reassign only
those that change from the default. This would benefit the economy and maintainability of complex configurations.

For simple values it is sufficient to reassign the new string. 

For complex variables, the branch must be "pruned" before redefining its structure: this is done by assigning the special value ```@``` to the complex variable: 

```
hooman
    version 1

<-- c:\hoomans\default.fud

company @
company
    managers
```

### Auto-increment by 1 

We have already encountered an example where complex variables are integers: 

```
hooman 
    version 1

company
    employees
        1
            name Johann Sebastian Bach
        2
            name Emmy Noether
        3
            name Pierre Louis Moreau de Maupertuis
```

If we wanted to reorganize the list to put it in alphabetical order, we would be forced to renumber the lines. 

Hooman offers a shortcut using the magic variable ```+```.

```
hooman 
    version 1

company
    employees
        +
            name Johann Sebastian Bach
        +
            name Emmy Noether
        +
            name Pierre Louis Moreau de Maupertuis
```

## Custom syntax

It is possible to define an object language by imposing constraints on both complex values (structure constraints) and simple values (string validation using regular expressions and javascript assignment rules).

### Constraints on complex values

Below ```hooman/syntax/structure``` path you can write the subdocument which is a mandatory template for what is outside the "hooman" branch.

```
hooman 
    version 1
    syntax
        structure
            company
                employees
                    id *
                        name !
                        info

company
    employees
        1
            name Johann Sebastian Bach
            info
                <<
                    Bach was born in 1685 in Eisenach, 
                    in the duchy of Saxe-Eisenach, into an 
                    extensive musical family.
                >> 
        2
            name Emmy Noether
```

The exclamation point ```!``` after the identifier means that the variable is mandatory if the branch exists.

The asterisk ```*``` allows flexibility in assigning the name of a variable: in the example shown, instead of id we can put numbers or names; this is indispensable in case of enumerations.

Hooman also supports recursive structures. Let the following example be given: 

```
hooman 
    version 1
    syntax
        structure
            root
                description
                sublevels ...
                    id *
                        description unknown
                        sublevels

root
    description primary level
    sublevels
        1
            description sublevel 1
            sublevels
                1
                    description sublevel 1.1
                2
                    description sublevel 1.2
        2
            description sublevel 2
                1
                    description sublevel 2.1
                2
                    description sublevel 2.2
                2
                    description sublevel 2.3
```

When the variable ```sublevels```, followed by the triple dot ```...``` is repeated in its sub-document, this indicates that the constraint must be applied recursively.

You can declare default values in the ```hooman/syntax/structure``` section by adding it to the 
right of the variable name.

In the previous example, the ```description unknown``` assignment is replicated under each node ```sublevels/*```; 
if, later, type were explicitly reassigned, the variable would assume the new explicit value. 

### Constraints on simple values

Below ```hooman/syntax/rules``` path you can list the rules on simple values. 
A rule consists of pre-conditions followed by post-conditions separated by the arrow symbol ```==>```.

A condition is a name-value pair in which the value is a regular expression: a string satisfies the condition if it is captured entirely by the regex.

The names of the pre-conditions must be identifiers in the branch that is about to be validated.

A rule is triggered if all pre-conditions are met. A rule with no pre-conditions is always active when the post-condition variable occurs.

If a rule is active, then all post-conditions must be satisfied.

In the following example, the ```type``` variable can only take the values ​​```string```, ```number``` and ```date```.

If *type = number*, then ```value``` can have a form like ```23.45```; if *type = date*, then ```value``` 
can have a form like ```2021-03-14```.

```
hooman 
    version 1
    syntax
        structure
            fields
                id *
                    type string
                    value
        rules
            1
                ==> 
                type (string|number|date)
            +
                type number
                ==>
                value \d+(\.\d+)?
            +
                type date
                ==>
                value (\d{4})-(\d{2})-(\d{2})

fields
    name
        value Johann Sebastian Bach

    amount
        type number
        value 23.45

    tradedate
        type date
        value 2021-03-14
```

In this case, a weak validation of the date is obtained: July 38 would be considered valid!

For strong validation of the date, you can use the guillemots tag:

```
hooman 
    version 2
    syntax
        ...
            +
                type date
                ==>
                value 
                    <<date[ymd]
                        (\d{4})-(\d{2})-(\d{2})
                    >>
        ...
```

You can also use the guillemots tag to inform of the numeric type to the recipient of the document:


```
hooman 
    version 2
    syntax
        ...
            +
                type number
                ==>
                value 
                    <<number[.2]
                        \d+(\.\d+)?
                    >>
        ...
```

## Assignment rules

You can use rules to assign values, not just to validate them. 
If the post-condition was a javascript formula, indicated by the "js" tag, then this formula will be used to value the occurrences of the variable in the document:

```
hooman 
    version 2
    syntax
        structure
            invoice
                items
                    item *
                        quantity
                        price
                        amount
        rules
            +
                type quantity
                ==>
                value 
                    <<number[.]
                        \d+
                    >>
            +
                type price
                ==>
                value 
                    <<number[.2]
                        \d+(\.\d+)?
                    >>
            +
                type amount
                ==>
                value 
                    <<js
                        branch.quantity * branch.price
                    >>

invoice
    items
        bolts
            quantity 100
            price    0.50
            amount
        screws
            quantity 50
            price    0.75
            amount
```

Thus, a hooman parser can calculate the values automatically: in this case it will calculate the "amount" variable.

To write the formula, some useful items are available:
* ```hoo```: a JSON containing the whole document (except the hooman branch);
* ```branch```: a JSON containing the siblings of the current variable;
* ```self```: the variable itself;
* ```path```: an array with the path from ```hoo``` (excluded) to ```branch```.

There's a "library" section for collect javascript functions useful to assignment rules:


```
hooman 
    version 2
    syntax
        structure
            ...
        rules
            ...
        library
            +
                <<js
                    function doubleUp(x){
                        return 2 * x;
                    }
                >>
            ...
```

## Security

The Hooman v2 specifications include some features to ensure security when a multi-document 
is written by multiple authors with different levels of clearance.

### Classifier

It's possible to specify an identifier that classifies the document as belonging 
to a typology using the variable placed as child of ```hooman```:

```
hooman 
    version 2
    classifier masterdata
    syntax
        structure
            ...
```


### Modules

In the ```syntax``` section, using the ```modules``` variable, you can list 
the file classifiers that you want to be included in the main document:

```
hooman 
    version 2
    classifier invoice
    syntax
        structure
            ...
        rules
            ...
        modules
	    syntax
            masterdata
			
```


### Locked

Usually the value of a variable can be overwritten. If you want to prevent data included by another file from being modified, you can use the variable ```locked``` in the included file; a write access by another document, main or secondary, raises an access violation exception:

```
hooman 
    version 2
    locked
    syntax
        structure
            ...
```
