# ucpp
A C preprocessor designed to be embeddable, quick, light and fully compliant to ISO Standard 9899:1999, aka ISO C99, or simply, C99

ucpp-1.3.2 is a C preprocessor compliant to ISO-C99.

Author: Thomas Pornin <pornin@bolet.org>
Main site: http://pornin.nerim.net/ucpp/



INTRODUCTION
------------

A C preprocessor is a part of a C compiler responsible for macro
replacement, conditional compilation and inclusion of header files.
It is often found as a stand-alone program on Unix systems.

ucpp is such a preprocessor; it is designed to be quick and light,
but anyway fully compliant to the ISO standard 9899:1999, also known
as C99. ucpp can be compiled as a stand-alone program, or linked to
some other code; in the latter case, ucpp will output tokens, one
at a time, on demand, as an integrated lexer.

ucpp operates in two modes:
-- lexer mode: ucpp is linked to some other code and outputs a stream of
tokens (each call to the lex() function will yield one token)
-- non-lexer mode: ucpp preprocesses text and outputs the resulting text
to a file descriptor; if linked to some other code, the cpp() function
must be called repeatedly, otherwise ucpp is a stand-alone binary.



INSTALLATION
------------

1. Uncompress the archive file and extract the source files.

2. Edit tune.h. Here is a short explanation of compile-time options:

  LOW_MEM
     Enable memory-saving functions; this is for low-end and old systems,
     but seems to be good for larger systems too. Keep it.
  NO_LIBC_BUF
  NO_UCPP_BUF
     Two options used to disable the two bufferings inside ucpp. Define
     both options for maximum memory savings but you will probably want
     to keep libc buffering for decent performance. Define none on large
     systems (modern 32 or 64-bit systems).
  UCPP_MMAP
     With this option, if ucpp internal buffering is active, ucpp will
     try to mmap() the input files. This might yield a slight performance
     improvement, but will work only on a limited set of architectures.
  PRAGMA_TOKENIZE
     Make ucpp generate tokenized PRAGMA tokens on #pragma and _Pragma();
     tokenization is made this way: tokens are assembled as a null
     terminated array of unsigned chars; if a token has a string value
     (as defined by the STRING_TOKEN macro), the value follows the token,
     terminated by PRAGMA_TOKEN_END (by default, a newline character cast
     to unsigned char). Whitespace tokens are skipped. The "name" value
     of the PRAGMA token is a pointer to that array. This setting is
     irrelevant in non-lexer mode.
  PRAGMA_DUMP
     In non-lexer mode, keep #pragma in output; non-void _Pragma() are
     translated to the equivalent #pragma. Irrelevant in lexer mode.
  NO_PRAGMA_IN_DIRECTIVE
     Do not evaluate _Pragma() inside #if, #include, #include_next and #line
     directives; instead, emit an error (since the remaining _Pragma will
     surely imply a syntax error).
  DSHARP_TOKEN_MERGE
     When two tokens are to be merged with the `##' operator, but fail
     because they do not merge into a single valid token, ucpp keeps those
     two tokens separate by adding an extra space between them in text
     output. With this option on, that extra space is not added, which means
     that some tokens may merge partially if the text output is preprocessed
     again. See tune.h for details.
  INMACRO_FLAG
     In lexer mode, set the inmacro flag to 1 if the current token comes
     from a macro replacement, 0 otherwise. macro_count maintains an
     increasing counter of such replacements. DT_CONTEXT tokens count as
     one macro replacement each. #pragma, and _Pragma() that do not come
     from a macro replacement, also count as one macro replacement each.
     This setting is irrelevant in non-lexer mode.
  STD_INCLUDE_PATH
     Default include path in stand-alone ucpp.
  STD_MACROS
     Default predefined macros in stand-alone ucpp.
  STD_ASSERT
     Default assertions in stand-alone ucpp.
  NATIVE_SIGNED
  NATIVE_UNSIGNED
  NATIVE_UNSIGNED_BITS
  NATIVE_SIGNED_MIN
  NATIVE_SIGNED_MAX
  SIMUL_ARITH_SUBTYPE
  SIMUL_SUBTYPE_BITS
  SIMUL_NUMBITS
  WCHAR_SIGNEDNESS
     Those options define how #if expressions are evaluated; see the
     cross-compilation section of this file for more info, and the
     comments in tune.h. Extra info is found in arith.h and arith.c,
     at the possible expense of your mental health.
  DEFAULT_LEXER_FLAGS
  DEFAULT_CPP_FLAGS
     Default flags in respectively lexer and non-lexer modes.
  POSIX_JMP
     Define this if your architecture defines sigsetjmp() and
     siglongjmp(); it is known to (very slightly) improve performance
     on AIX systems.
  MAX_CHAR_VAL
     ucpp will consider characters whose value is equal or above
     MAX_CHAR_VAL as outside the C source charset (so they will be
     treated just like '@', for instance). For ASCII systems, 128
     is fine. 256 is a safer value, but uses more (static) memory.
     For performance reasons, use a power of two. If MAX_CHAR_VAL is
     correctly adjusted, ucpp should be compatible with any character
     set.
  UNBREAKABLE_SPACE
     If you want an extra-whitespace character, define this macro to that
     character. For instance, define this to 160 on an ISO-8859-1 system
     if you want the 'unbreakable space' to be considered as whitespace.
  SEMPER_FIDELIS
     With this option set, ucpp, when used as a lexer, will pass
     whitespace tokens to its caller, and those tokens will have their
     true content; this is intended for reconstruction of the source
     line. Beware that some comments may have embedded newlines.
  COPY_LINE_LENGTH
     ucpp can maintain a copy of the current source line, up to that
     length. Irrelevant to stand-alone version.
  *_MEMG
     Those settings modify ucpp behaviour, wrt memory allocations. With
     higher values, ucpp will perform less malloc() calls and will run
     faster, but it will use more memory. Reduce INPUT_BUF_MEMG and
     OUTPUT_BUF_MEMG on low-memory systems, if you kept ucpp buffering
     (see NO_UCPP_BUF option).

3. Edit the Makefile. You should define the variables CC and FLAGS;
   there are the following options:

  -DAUDIT
     Enable internal sanity checks; this slows down a bit ucpp. Do not
     define unless you plan to debug ucpp.
  -DMEM_CHECK
     With this setting, ucpp will check for the return value of malloc()
     and exit with a diagnostic when out of memory. MEM_CHECK is implied
     by AUDIT.
  -DMEM_DEBUG
     Enable memory debug code. This will track memory leaks and several
     occurrences of memory management errors; it will also slow down
     things and increase memory consumption, so you probably do not
     want to use this option.
  -DINLINE=foobar
     The ucpp code uses "inline" qualifier for some functions; by
     default, that qualifier is macro-replaced with nothing. Define
     INLINE to the correct replacement for your compiler, if supported.
     Note that all "inline" functions in ucpp are also "static". For any
     C99-compliant compiler, the GNU compiler (gcc), and the Compaq C
     compiler under Linux/Alpha, no -DINLINE is needed (see tune.h for
     details).

4. Compile by typing "make". This should produce the ucpp executable
   file. You might see some warning messages, especially with gcc:
   gcc believes some variables might be used prior to their
   initialization; ignore those messages.

5. Install wherever you want the binary and the man page ucpp.1. I
   have not provided an install sequence because I didn't bother.

6. If you do not have the make utility, compile each file separately
   and link them together. The exact details depend on your compiler.
   You must define the macro STAND_ALONE when compiling cpp.c (there
   is such a definition, commented out, in cpp.c, line 34).

There is no "configure" script because:
-- I do not like the very idea of a "configure" script.
-- ucpp is written in ANSI-C and should be fairly portable.
-- There is no such thing as "standard" settings for a C preprocessor.
   The predefined system macros, standard assertions,... must be tuned
   by the sysadmin.
-- The primary goal of ucpp is to be included in compilers. The
   stand-alone version is mainly a debugging tool.

Please note that you need an ISO-C90 (formerly ANSI) C compiler suite
(including the standard library) to compile ucpp. If your compiler is
not C99 (or later), read the cross-compilation section in this README
file.

The C90 and C99 standards state that external linkage names might be
considered equal or different based upon only their first 6 characters;
this rule might make ucpp not compile on a conformant C implementation.
I have yet to see such an implementation, however.

If you want to use ucpp as an integrated preprocessor and lexer, see the
section REUSE. Compiling ucpp as a library is an exercise left to the
reader.

With the LOW_MEM code enabled, ucpp can run on a Minix-i86 or Msdos
16-bit small-memory-model machine. It will not be fully compliant
on such an architecture to C99, since C99 states that at least one
source code with 4095 simultaneously defined macros must be processed;
ucpp will be limited to about 1500 macros (at most) due to memory
restrictions. At least ucpp can preprocess its own code in these
conditions. LOW_MEM is on by default because it seems to improve
performance on large systems.



LICENSE
-------

The copyright notice and license is at the beginning of the Makefile and
each source file. It is basically a BSD license, without the advertising
subclause (which BSD dropped recently anyway) and with no reference to
Berkeley (since the code is all mine, written from scratch). Informally,
this means that you can reuse and redistribute the code as you want,
provided that you state in the documentation (or any substantial part of
the software) of redistributed code that I am the original author. (If
you press a cdrom with 200 software packages, I do not insist on having
my name on the cover of the cdrom -- just keep a Readme file somewhere
on the cdrom, with the copyright notice included.)

As a courteous gesture, if you reuse my code, please drop me a mail.
It raises my self-esteem.



REUSE
-----

The code has been thought as part of a bigger project; it might be
used as an integrated lexer, that will read files, process them as a
C preprocessor, and output a stream of C tokens. To include this code
into a project, compile with STAND_ALONE undefined.

To use the preprocessor and lexer, several steps should be performed.
See the file 'sample.c' for an example.

1. call init_cpp(). This function initializes the lexer automaton.

2. set the following global variables:
	no_special_macros
		non-zero if the special macros (__FILE__ and others)
		should not be defined. This is a global flag since
		it affects the redefinition of such macros (which are
		allowed if the special macros are not defined)
	c99_compliant
		if non-zero, define __STDC_VERSION__ to 199901L; this
		is the default; otherwise, do not define __STDC_VERSION__.
		Note that ucpp will accept to undefine __STDC_VERSION__
		with a #undef directive.
	c99_hosted
		if strictly positive, define __STDC_HOSTED__ to 1.
		If zero, define __STDC_HOSTED__ to 0. If negative,
		do not define __STDC_HOSTED__. The default is 1.
	emit_defines and emit_assertions should be set to 0 for
	the step 3.

3. call init_tables(). This function initializes the macro table
   and other things; it will intialize assertions if it has a non-zero
   argument.

4. call init_include_path(). This function will reset the include
   path to the list of paths given as argument.

5. set the following global variables
	emit_dependencies
		set to 1 if dependencies should be emitted during
		preprocessing
		set to 2 if dependencies should also be emitted for
		system include files
	emit_defines
		set to non-zero if #define macro definitions should be
		emitted when macros are defined
	emit_assertions
		set to non-zero if #define macro definitions should be
		emitted when macros are defined
	emit_output
		the FILE * where the above items are sent if one of the
		three emit_ variables is set to non zero
	transient_characters
		this is for some cross-compilation; see the relevant
		part in this README file for details

6. call set_init_filename() with the initial filename as argument;
   the second argument indicates whether the filename is real or
   conventional ("real" means "an fopen() on it will work").

7. initialize your struct lexer_state:
	call init_lexer_state()
	call init_lexer_mode() if the preprocessor is supposed to
	   output a list of tokens, otherwise set the flags field
	   to DEFAULT_CPP_FLAGS and set the output field to the
	   FILE * where output should be sent
	(init_lexer_mode(), if called at all, must be called after
	 init_lexer_state())
	adjust the flags field; here is the meaning of flags:

WARN_STANDARD
	emit the standard warnings
WARN_ANNOYING
	emit the useless and annoying warnings
WARN_TRIGRAPHS
	count trigraphs encountered; it is up to the caller to emit
	a warning if some trigraphs were indeed encountered; the count
	is stored in the count_trigraphs field of the struct lexer_state
WARN_TRIGRAPHS_MORE
	emit a warning for each trigraph encountered
WARN_PRAGMA
	emit a warning for each non-void _Pragma encountered in non-lexer
	mode (because these are dumped as #pragma in the output) and for each
	#pragma too, if ucpp was compiled without PRAGMA_DUMP
FAIL_SHARP
	emit errors on '#' tokens beginning a line and not followed
	by a valid cpp directive
CCHARSET
	emit errors when non-C characters are encountered; if this flag
	is not set, each non-C character will be considered as a BUNCH
	token (since C99 states that non-C characters are allowed as
	long as they "disappear" during preprocessing [through macro
	replacement and stringification for instance], this flag must
	not be set, for maximum C99 compliance)
DISCARD_COMMENTS
	do not keep comments in output (irrelevant in lexer mode)
CPLUSPLUS_COMMENTS
	understand new style comments (//) (mandatory for C99)
LINE_NUM
	emit #line directives when entering a file, if not in lexer mode;
	emit DT_CONTEXT token in lexer mode for #line and new files
GCC_LINE_NUM
	if LINE_NUM is set, emit gcc-like directives instead of #line
HANDLE_ASSERTIONS
	understand assertions in #if expressions (and #assert, #unassert)
HANDLE_PRAGMA
	make PRAGMA tokens for #pragma; irrelevant in non-lexer mode
	(handling of some pragmas is required in C99 but is not of
	the competence of the preprocessor; without this flag, ucpp will
	ignore the contents of #pragma and _Pragma directives)
MACRO_VAARG
	understand macros with a variable number of arguments (mandatory
	for C99)
UTF8_SOURCE
	understand UTF-8 encoding: multibyte characters are considered
	equivalent to letters as far as syntax is concerned (they can
	be used in identifiers)
LEXER
	act as a lexer, outputting tokens
TEXT_OUTPUT
	this flag should be set to 0 if ucpp works as a lexer, 1 otherwise.
	It is somehow redundant with the LEXER flag, but the presence of
	those two different flags is needed in ucpp.
KEEP_OUTPUT
	in non-lexer mode, emit the result of preprocessing
COPY_LINE
	maintain a copy of the last read line in the copy_line field of
	the struct lexer_state ; see below for how to use this buffer
HANDLE_TRIGRAPHS
	understand trigraphs, such as ??/ for \. This option should be
	set by default, except for some legacy code.

	There are other flags, but they are for private usage of ucpp.

8. adjust the input field in the lexer_state to the FILE * from where
   source file is read. If you use the UCPP_MMAP compile-time option,
   and your input file is eligible to mmap(), then you can call
   fopen_mmap_file() to open it, then set_input_file() to set ls->input
   and some other internal options. Do not call set_input_file() unless
   you just called fopen_mmap_file() just before on the same file.

9. call add_incpath() to add an include path, define_macro() and
   undef_macro() to add or remove macros, make_assertion() and
   destroy_assertion() to add or remove assertions.

10. call enter_file() (this is needed only in non-lexer mode, or if
    LINE_NUM is set).


Afterwards:

-- if you are in lexer mode, call lex(); each call will make the ctok
   field point to the next token. A non-zero return value is an error.
   lex() skips whitespace tokens. The memory used by the string value
   of some tokens (identifiers, numbers...) is automatically freed,
   so copy the contents of each such token if you want to keep it
   (tokens with a string content are identified by the STRING_TOKEN
   macro applied to their type).
   When lex() returned a non-zero value: if it is CPPERR_EOF, then
   end-of-input was reached. Otherwise, it is a genuine error and
   ls->ctok is an undefined token; skip it and call lex() again to
   ignore the error.

-- otherwise, call cpp(); each call will analyze one or more tokens
   (one token if it did find neither a cpp directive nor a macro name).
   A positive return value is an error.

For both functions, if the return value is CPPERR_EOF (which is a
strictly positive value), then it means that the end of file was
reached. Call check_cpp_errors() after end of file for pending errors
(unfinished #if constructions for instance). In non-lexer mode,
call flush_output().

In the struct lexer_state, the following fields might be read:
	line		   the current input line number
	oline		   the current output line number (in non-lexer mode)
	flags		   the flags described above
	count_trigraphs	   the number of trigraphs encountered
	inmacro		   the current token comes from a macro
	macro_count	   the current macro counter
"flags" is an unsigned long and might be modified; the three others
are of long type.


To perform another preprocessing: use free_lexer_state() to release
memory used by the buffers referenced in lexer_state, and go back to
step 2. The different tables (macros, assertions...) should be reset to
their respective initial contents.

There is also the wipeout() function: when called, it should release
(almost) all memory blocks allocated dynamically. After a wipeout(),
ucpp should be back to its state at step 2 (init_cpp() initializes only
static tables, that are never freed nor modified afterwards).


The COPY_LINE buffer: the struct lexer_state contains two interesting
fields, copy_line[] and cli. If the COPY_LINE flag is on, each read
line is stored in this buffer, up to (at most) COPY_LINE_LENGTH - 1
characters (COPY_LINE_LENGTH is defined in tune.h). The last character
of the buffer is always a zero, and if the line was read entirely, it is
zero terminated; the trailing newline is not included.

The purpose of this buffer is error-reporting. When an error occurs
(cpp() returns a strictly positive value, or lex() returns a non-zero
value), if your struct lexer_state is called ls, use this code:

	if (ls.cli != 0) ls.copy_line[ls.cli] = 0;

This will add a trailing 0 if the line was not read entirely.


ucpp may be configured at runtime to accept alternate characters as
possible parts of identifiers. Typical intended usage is for the '$'
and '@' characters. The two relevant functions are set_identifier_char()
and unset_identifier_char(). When this call is issued:
	set_identifier_char('$');
then for all the remaining input, the '$' character will be considered
as just another letter, as far as identifier tokenizing is concerned. This
is for identifiers only; numeric constants are not modified by that setting.
This call resets things back:
	unset_identifier_char('$');
Those two functions modify the static table which is initialized by
init_cpp(). You may call init_cpp() at any time to restore the table
to its standard state.

When using this feature, take care of the following points:

-- Do NOT use a character whose numeric value (as an `unsigned char'
cast into an `int') is greater than or equal to MAX_CHAR_VAL (in tune.h).
This would lead to unpredictable results, including an abrupt crash of
ucpp. ucpp makes absolutely no check whatsoever on that matter: this is
the programmer's responsibility.

-- If you use a standard character such as '+' or '{', tokens which
begin with those characters cease to exist. This can be troublesome.
If you use set_identifier_char() on the '<' character, the handling of
#include directives will be greatly disturbed. Therefore the use of any
standard C character in set_identifier_char() of unset_identifier_char()
is declared unsupported, forbidden and altogether unwise.

-- Stricto sensu, when an extra character is declared as part of an
identifier, ucpp behaviour cease to conform to C99, which mandates that
characters such as '$' or '@' must be treated as independant tokens of
their own. Therefore, if your purpose is to use ucpp in a conformant
C implementation, the use of set_identifier_char() should be made at
least a runtime option.

-- When enabling a new character in the middle of a macro replacement,
the effect of that replacement may be delayed up to the end of that
macro (but this is a "may" !). If you wish to trigger this feature with
a custom #pragma or _Pragma(), you should remember it (for instance,
usine _Pragma() in a macro replacement, and then the extra character
in the same macro replacement, is not reliable).



COMPATIBILITY NOTES
-------------------

The C language has a lengthening history. Nowadays, C comes in three
flavours:

-- Traditional C, aka "K&R". This is the language first described by
Brian Kernighan and Dennis Ritchie, and implemented in the first C
compiler that was ever coded. There are actually several dialects of
K&R, and all of them are considered deprecated.

-- ISO 9899:1990, aka C90, aka C89, aka ANSI-C. Formalized by ANSI
in 1989 and adopted by ISO the next year, it is the C flavour many C
compilers understand. It is mostly backward compatible with K&R C, but
with enhancements, clarifications and several new features.

-- ISO 9899:1999, aka C99. This is an evolution on C90, almost fully
backward compatible with C90. C99 introduces many new and useful
features, however, including in the preprocessor.

There was also a normative addendum in 1995, that added a few features
to C90 (for instance, digraphs) that are also present in C99. It is
sometimes refered to as "C95" or "AMD 1".


ucpp implements the C99 standard, but can be used in a stricter mode,
to enforce C90 compatibility (it will, however, still recognize some
constructions that are not in plain C90).

ucpp also knows about several extensions to C99:

-- Assertions: this is an extension to the defined() operator, with
   its own namespace. Assertions seem to be used in several places,
   therefore ucpp knows about them. It is recommended to enable
   assertions by default on Solaris systems.
-- Unicode: the C99 norm specifies that extended characters, from
   the ISO-10646 charset (aka "unicode") can be used in identifiers
   with the notations \u and \U. ucpp also accepts (with the proper
   flag) the UTF-8 encoding in the source file for such characters.
-- #include_next directive: it works as a #include, but will look
   for files only in the directories specified in the include path
   after the one the current file was found. This is a GNU-ism that
   is useful for writing transparent wrappers around header files.

Assertions and unicode are activated by specific flags; the #include_next
support is always active.

The ucpp code itself should be compatible with any ISO-C90 compiler.
The cpp.c file is rather big (~ 64kB), it might confuse old 16-bit C
compilers; the macro.c file is somewhat large also (~ 47kB).

The evaluation of #if expressions is subject to some subtleties, see the
section "cross-compilation".

The lexer code makes no assumption about the source character set, but
the following: source characters (those which have a syntactic value in
C; comment and string literal contents are not concerned) must have a
strictly positive value that is strictly lower than MAX_CHAR_VAL. The
strict positivity is already assured by the C standard, so you just need
to adjust MAX_CHAR_VAL.

ucpp has been tested succesfully on ASCII/ISO-8859-1 and EBCDIC systems.
Beware that UTF-8 is NOT compatible with EBCDIC.

Pragma handling: when used in non-lexer mode, ucpp tries to output a
source text that, when read again, will yield the exact same stream of
tokens. This is not completely true with regards to line numbering in
some tricky macro replacements, but it should work correctly otherwise,
especially with pragma directives if the compile-time option PRAGMA_DUMP
was set: #pragma are dumped, non-void _Pragma() are converted to the
corresponding #pragma and dumped also.

ucpp does not macro-replace the contents of #pragma and _Pragma();
If you want a macro-replaced pragma, use this:

#define pragma_(x)	_Pragma(#x)
#define pragma(x)	pragma_(x)

Anyway, pragmas do not nest (an _Pragma() cannot be evaluated if it is
inside a #pragma or another _Pragma).


I wrote ucpp according to what is found in "The C Programming Language"
from Brian Kernighan and Dennis Ritchie (2nd edition) and the C99
standard; but I could have misinterpreted some points. On some tricky
points I got help from the helpful people from the comp.std.c newsgroup.
For assertions and #include_next, I mimicked the behaviour of GNU cpp,
as is stated in the GNU cpp info documentation. An open question is
related to the following code:

#define undefined	!
#define makeun(x)	un ## x
#if makeun(defined foo)
qux
#else
bar
#endif

ucpp will replace 'defined foo' with 0 first (since foo is not defined),
then it will replace the macro makeun, and the expression will become
'un0', which is replaced by 0 since this is a remaining identifier. The
expression evaluates to false, and 'bar' is emitted.
However, some other preprocessors will replace makeun first, considering
that it is not part of a 'defined' operator application; this will
produce the macro 'undefined', which is replaced, and the expression
becomes '!foo'. 'foo' is replaced by 0, the expression evaluates to
true, and 'qux' is emitted.

My opinion is that the behaviour is undefined, because use of the
'defined' operator does not match an allowed form prior to macro
replacement (I mean, its syntax matches, but its use is reconverted
to inexistant and therefore is not anymore matching). Other people
think that the behaviour is well-specified, and contrary to what ucpp
does. The only thing clear to me is that the wording of the standard
(paragraph 6.10.1.3) is unclear.

Since the ucpp behaviour makes ucpp code simpler and cleaner, and
that it is unlikely that any real-life code would ever be disturbed
by that interpretation of the standard, ucpp will keep its current
behaviour until convincing evidence of my misinterpretation of the
standard is given to me. The problem can only occur if one uses ## to
make a 'defined' operator disappear from a #if expression (everybody
agrees that the generation of a 'defined' operator triggers undefined
behaviour).


Another point about macro replacement has been discussed at length in
several occasions. It is about the following code:

#define CAT(a, b)    CAT_(a, b)
#define CAT_(a, b)   a ## b
#define AB(x, y)     CAT(x, y)
CAT(A, B)(X, Y)

ucpp will produce `CAT(X,Y)' as replacement for the last line, whereas
some other preprocessors output `XY'. The answer to the question
"which behaviour is correct" seems to be "this is not defined by the
C standard". It is the answer that has been actually given by the C
standardization committee in 1992, to the defect report #017, question
23, which asked that very same question. Since the wording of the
standard has not changed in these parts from the 1990 to the 1999
version, the preprocessor behaviour on the above-stated code should
still be considered as undefined.

It seems, however, that there used to be a time (around 1988) when the
committee members agreed upon a precise macro-replacement algorithm,
which specified quite clearly the preprocessor behaviour in such
situation. ucpp behaviour is occasionnaly claimed as "incorrect" with
regards to that algorithm. Since that macro replacement algorithm has
never been published, and the committee itself backed out from it in
1992, I decided to disregard those feeble claims.

It is possible, however, that at some point in the future I rewrite the
ucpp macro replacement code, since that code is a bit messy and might be
made to use less memory in some occasions. It is then possible that, in
the aftermath of such a rewrite, the ucpp behaviour for the above stated
code become tunable. Don't hold your breath, though.


About _Pragma: the standard is not clear about when this operator is
evaluated, and if it is allowed inside #if directives and such. For
ucpp, I coded _Pragma as a special macro with lazy replacement: it will
be evaluated wherever a macro could be replaced, and only at the end of
the macro replacement (for practical purposes, _Pragma can be considered
as a macro taking one argument, and being replaced by nothing, except
for some tricky uses of the # and ## operators). This means that, by
default, ucpp will evaluate _Pragma inside some directives (mainly, #if,
#include, #include_next and #line), but it can be taught not to do so by
defining NO_PRAGMA_IN_DIRECTIVE in tune.h.



CROSS-COMPILATION
-----------------

If compiled with a C99 development suite, ucpp should be fully
C99-compliant on the host platform (up to my own understanding of the
standard -- remember that this software is distributed as-is, without
any guarantee). However, if a pre-C99 compiler is used, or if the
target machine is not the host machine (for instance when you build a
cross-compiler), the evaluation of #if expressions is subject to some
cross-compiling issues:


-- character constants: when evaluating expressions, character constants
are interpreted in the source character set context; this is allowed
by the standard but this can lead to problems with code that expects
this interpretation to match the one made in the C code. To ease
cross-compilation, you can define a conversion array, and make the
global variable transient_characters point to it. The array should
contain 256 int; transient_characters[x] is the value of the character
whose value is x in the source character set.

This facility is provided for inclusion of ucpp inside another code;
if you want a stand-alone ucpp with that conversion, hard-code the
conversion table into eval.c and make transient_characters[] statically
point to it. Alternatively, you could provide an option syntax to
provide such a table on command-line, if you feel like it.


-- wide character constants signedness: by default, ucpp makes wide
characters as signed as what plain chars are on the build host. To
force wide character constant signedness, define WCHAR_SIGNEDNESS to 0
(for unsigned) or 1 (for signed). Beware, however, that "native" wide
character constants, even signed, are considered positive. Non-wide
character constants are, according to the C99 standard, of type int, and
therefore always signed.


-- evaluation type: C90 states that all constants in #if expressions
are considered as either long or unsigned long, and that the evaluation
is performed with operands of that size. In C99, the situation is
equivalent, except that the types used are intmax_t and uintmax_t, as
defined in <stdint.h>.

ucpp can use two expression evaluators: one uses native integer types
(one signed and one unsigned), the other evaluator emulates big integer
numbers by representing them with two values of some unsigned type. The
emulated type handles signed values in two's complement representation,
and can be any width ranging from 2 bits to twice the size of the
underlying native unsigned type used. An odd width is allowed. When
right shifting an emulated signed negative value, it is left-padded with
bits set to 1 (this is sign extension).

When the ARITHMETIC_CHECKS macro is defined in tune.h, all occurrences
of implementation-defined or undefined behaviour during arithmetic
evaluation are reported as errors or warned upon. This includes all
overflows and underflows on signed quantities, constants too large,
and so on. Errors (which terminate immediately evaluation) are emitted
for division by 0 (on / and % operators) and overflow (on / operator);
otherwise, warnings are emitted and the faulty evaluation takes place.
This prevents ucpp from crashing on typical x86 machines, while still
allowing to use some extensions.



FUTURE EVOLUTIONS
-----------------

ucpp is quite complete now. There was a longstanding project of
"traditional" preprocessing, but I dropped it because it would not
map cleanly on the token-based ucpp structure. Maybe I will code a
string-based preprocessor one day; it would certainly use some of the
code from lexer.c, eval.c, mem.c and nhash.c. However, making such a
tool is almost irrelevant nowadays. If one wants to handle such project,
using ucpp as code base, I would happily provide some help, if needed.



CHANGES
-------

From 1.2 to 1.3:

* brand new integer evaluation code, with precise evaluation and checks
* new hash table implementation, with binary trees
* relaxed attitude on failed `##' operators
* bugfix on macro definition on command-line wrt nesting macros
* support for up to 32766 macro arguments in LOW_MEM code
* support for optional additional "identifier" characters such as '$' or '@'
* bugfix: memory leak on void #assert

From 1.1 to 1.2:

* bugfix: numerous memory leaks
* new function: wipeout(); this should release all malloc() blocks
* bugfix: missing "newline" and trailing "context" tokens
* improved included files name caching
* included memory leak detection code

From 1.0 to 1.1:

* bugfix: missing newline when exiting from a non-newline-terminated file
* bugfix: crash when resetting due to definition of the _Pragma pseudo-macro
* bugfix: handling of additional "optional" whitespace with SEMPER_FIDELIS
* improved handling of unreplaced arg macros wrt output line
* tricky handling of utterly tricky #include
* bugfix: spurious token `~=' eliminated

From 0.9 to 1.0:

* bugfix: crash after erroneous #assert
* changed ERR_SHARP to FAIL_SHARP, EMUL_UINTMAX to SIMUL_UINTMAX
* made "inline" default on gcc and DEC ccc (Linux/Alpha)
* semantic of -I is now Unix-like (added directories are looked first)
* added -J flag (to add include directories after the system ones)
* cleaned up non-ascii issues
* bugfix: missing brace in no-LOW_MEM code
* bugfix: argument number check in variadic macros
* bugfix: crash in non-lexer mode after some cases of unreplaced macro
* bugfix: _Pragma() handling wrt # and ##
* made evaluation of _Pragma() optional in #if, #include and #line
* bugfix: re-dump of multiline #pragma
* added the inmacro and macro_count flags
* added mmap() support
* added option to retain whitespace content in lexer mode

From 0.8 to 0.9:

* added check for division by 0 in #if evaluation
* added check for non-standard line numbers
* added check for trailing garbage in most directives
* corrected signedness of char constants (always int, therefore always signed)
* made LOW_MEM code, so that ucpp runs smoothly on low memory architectures
* multiple bugfixes (using the GNU cpp testsuite)
* added handling of _Pragma (as a macro)
* added tokenization of pragma directives
* added conservation of pragma directives in text output
* produced Msdos 16-bit small memory model executable
* produced Minix-86 executable

From 0.7 to 0.8:

* added some support for Amiga systems
* fixed extra spacing in stringified tokens
* fixed bug related to %:% and tolerated rogue sharps
* namespace cleanup
* bugfix for macro redefinition
* added warning for evaluated comma operators in #if (ISO requirement)
* -Dfoo now defines foo with content 1 (and not void content)
* trigraphs can be disabled (for incorrect but legacy code)
* fixed semantics for #include "file" (local directory)
* fixed detection of protected files
* produced a Msdos 16-bit executable

From 0.6 to 0.7:

* officially changed the goal to full C99 compliance
* added the CONTEXT token and let NEWLINE tokens go
* added report_context() for error reporting
* enforced matching of #if/#endif (file-global nesting level = 0)
* added support of C99 digraphs
* added UTF-8 encoding support
* added universal character names
* rewrote #if expressions (sizes fixed, bignum, signed/unsigned fixed)
* fixed incomplete evaluation of #if expressions
* added transient_characters[]

From 0.5 to 0.6:

* disappearance of error_nonl()
* added extra optional warnings for trigraphs
* some bugfixes, especially in lexer mode
* handled MacIntosh files correctly

From 0.4 to 0.5:

* nicer #pragma handling (a token can be emitted)
* bugfix in lexer mode after #line and #error
* sample.c   an example of code linked with ucpp
* made #if expressions conforming to standard signed/unsigned handling
* added the copy_line[] buffer feature

From 0.3 to 0.4:

* relaxed interpretation of '#include foo' when foo ends up, after macro
  substitution, with a '<bar>' content
* corrected the 'double-dot' bug
* corrected two bugs related to the treatment of macro aborted calls (due
  to lack of arguments)
* some namespaces cleanup, to ease integration into other code
* documented the way to include ucpp into another program
* made newlines embedded into strings illegal (and reported as such)

From 0.2 to 0.3:

* added support for system predefined macros
* made several bugfixes
* checked C99 compliance for most of the features
* ucpp now accepts non-C characters on standard when used stand-alone
* removed many useless spaces in the output

From 0.1 to 0.2:

* added support for assertions
* added support for macros with variable arguments
* split the pharaonic cpp.c file into many
* made several bugfixes
* relaxed the behaviour with regards to the void arguments
* made C++-like comments an option



THANKS TO
---------

Volker Barthelmann, Neil Booth, Stephen Davies, Stephane Ecolivet,
Marc Espie, Marcus Holland-Moritz, Antoine Leca, Cyrille Lefevre,
Dave Rivers, Loic Tortay and Laurent Wacrenier, for suggestions and
beta-testing.

Paul Eggert, Douglas A. Gwyn, Clive D.W. Feather, and the other guys from
comp.std.c, for explanations about the standard.

Dave Brolley, Jamie Lokier and Neil Booth, for discussion about tricky
points on nesting macros.

Brian Kernighan and Dennis Ritchie, for bringing C to mortal Men.
