# Flex

[Flex](http://flex.sourceforge.net) is an open-source generator of lexical analyzers, also called scanners.

The scanners can parse a data stream (file or buffered string), recognize patterns, and execute actions when this happens.

This tutorial is an introduction to this tool.

### Input file
In order to generate the scanner (a C program), Flex needs an input file defining the rules the scanner will operate with. Flex rule files use the .lpp extension. A file is divided in three sections:

```
definitions
%%
rules
%%
code
```

On top of the file you can also add _top blocks_; the code there will be executed before anything else in the scanners' code:

```
%top {
  // all code here executed first
}
```

Top blocks usually contain preprocessor directives.

### Definitions

You usually start by defining options that control the behaviour of the Flex scanner. Here are some common options:

```%option header­file="header.h" ```
creates and header file.

```%option noyywrap```
the scanner stops at the EOF of the current input, i.e. it assumes there is nothing more to parse.

```%option outfile="scanner.c"```
Sets the scanner name.

```%option warn``` enables warnings

Sometimes it's useful to associate regular expressions used in the scanner with names easy to remember, so that the generated code is more readable.

Example:

```DIGIT [0­9]``` Every use of {DIGIT} inside a rule is replaced with [09].

### Rules
This is the heart of the scanner. The standard syntax is:

```
regexp  { action }
```

Flex supports an extended set of regular expressions; you can find a reference on the website.

When Flex matches a pattern in the stream, the match is copied in a variable, ```char *yytext```.
The length is contained in ```yyleng```.

The action, which is a regular C code snippet, is executed. The action part should be separated by the regexp by at least one whitespace.

If no action matches a given pattern, all the occurrences of that pattern will be deleted from the output.

Everything that does not match any pattern is printed in the output as it is (default rule).
There are special instructions that can be used in actions:

- ```ECHO```: prints out yytext (the match);
- ```input()```: reads the next character in the stream;
- ```yyterminate()```: exits with success.

### Start conditions

Start conditions allow you to define "states" inside the scanner, and directive such as "execute this action if there is a match and the scanner is in this state". Your scanner can then become a state machine.
All the conditions must be declared in the definition section:

```
%s cond1 cond2
```

With

```
<cond1>pattern1 {action1;}
```
action1 is executed only if, when the scanner found pattern1, it was in that condition. At the beginning,
a scanner is in the INITIAL condition. Then:

```
pattern {BEGIN(cond1);}
```

moves the scanner in condition cond1 when the pattern is matched. You can also write ```BEGIN(INITIAL)``` to go back to the INITIAL state.

### Buffers
The scanner reads its tokens from stdin. If we want to trigger a parsing, we can make the scanner read from
a file:

```
yyset_in(fpointer);
int ret = yylex();
```

Don't forget to include the header if these instructions are in another file. Other utilities include:

```
YY_BUFFER_STATE yy_create_buffer(FILE *f, int size);
```

creates a buffer file with the given size (if unsure, use YY_BUF_SIZE).

```
void yy_delete_buffer (YY_BUFFER_STATE buf);
```
deallocates the buffer.

```
YY_BUFFER_STATE yy_scan_string(const char *s);
```
scans a string.
