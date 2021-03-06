fnc command line reference
--------------------------

::

        fnc [options] [arguments]

*fnc* is the command line tool to interact with efene, it's a simple and
unified way to use efene, this tool allows to:

 * compile one or multiple files
 * run a program
 * start the interactive shell
 * debug all stages of compilation

This tool is just a thin front end that calls the appropiate erlang or efene
program, you can even use it to run compiled erlang programs.

Running The Interactive Shell
:::::::::::::::::::::::::::::

To start the interactive shell just type in your shell::

        fnc -s

Compiling one or more files
:::::::::::::::::::::::::::

To compile one or more files just give the filenames as parameters to fnc::

        # just one file
        fnc file1.fn

        # multiple files
        fnc file1.fn file2.ifn file3.fn

Running a compiled program
::::::::::::::::::::::::::

To run a program you have to pass the name of the module and function that you
want to run to *fnc* with the *-r* (run) flag::

        fnc -r mymodule run

That will run the *run* function from the *mymodule* module.

You can pass parameters to the function adding more values after the function name::

        fnc -r mymodule run parameter 1 two false

That call is equivalent to the call::

        mymodule.run(["parameter", "1", "two", "false"])

As you can see all parameters are passed as a list of strings.

Since efene and erlang programs have the same byte code you can run erlang
programs using *fnc*::

        # compile a program using the erlang compiler
        erlc mymodule.erl

        # run the program using fnc
        fnc -r mymodule run

Adding paths to look for modules
::::::::::::::::::::::::::::::::

When you start a program or start the shell you can instruct efene to add some
extra paths to look for modules.

To do this there are two options that add paths:

*-a <path>*
        append the path to the path list

*-p <path>*
        prepend the path to the path list

An example of this, imagine you have a directory with you modules in /home/user/lib
and you want to start the shell with that path so you can access those modules::

        fnc -s -a /home/user/lib

Now imagine that in /home/user/lib you have a newer version of a module that is
already available in the path where erlang looks for modules.

Adding the path to the end will make the other module to load instead of the one located at
/home/user/lib.

To load your version first you have to prepend your module (insert at the beggining)::

        fnc -s -p /home/user/lib

You can use this options as much as you need for example::

        fnc -r server start -p /home/user/lib -a mods -p libs -a modules

Transform a module to erlang
::::::::::::::::::::::::::::

To transform an efene or ifene module to erlang source code run::

        fnc -t erl module.fn

        fnc -t erl module.ifn

It will print the erlang source code to the screen, you can save it::

        fnc -t erl module.fn > module.erl

When Something goes wrong
:::::::::::::::::::::::::

you can see what *fnc* actually wanted to run passing the *-T* option as first
option.

This will make *fnc* to print the command instead of running it, here are some
examples::

        $ fnc -T -s
        erl -run shell -run init stop -noshell -pa "/home/mariano/Proyectos/efene/ebin"

        $ fnc -T foo.ifn
        erl -run fn run beam "." foo.ifn  -run init stop -noshell -pa "/home/mariano/Proyectos/efene/ebin"

        $ fnc -T foo.ifn bar.fn
        erl -run fn run beam "." foo.ifn bar.fn  -run init stop -noshell -pa "/home/mariano/Proyectos/efene/ebin"

        $ fnc -T -r mymodule run parameter 1 two false
        erl -run mymodule run parameter 1 two false  -run init stop -noshell -pa "/home/mariano/Proyectos/efene/ebin"

Advanced options
::::::::::::::::

.. note:
        
        This section is for advanced users, you don't need to know this to code or use efene.

efene transforms the source code into multiple data structures while it's
compiling, to make my life easier I added some options to debug the
intermediate representations of a module from the command line.

This tools can be also be used by people interested in helping efene or people
wanting to know more about erlang internals.

The different stages from a file to bytecode are the following::

        source -> lex -> tree -> ast -> mod -> beam

source

        the source code file you edit

lex
        
        a list of tuples that contains the *tokens* of a program, each token is
        represented by a two or three item tuple where the first item is the
        type of the token, the second item is the line where the token was
        found and the third (optional) item is the string representing the
        token.

        Some examples::

                {open,33,'('}, {atom,33,to},

        Those two tokens represent the string **(to** in the line 33 of a file.

tree

        a tree representing the Abstract Syntax Tree of efene, this is a tree
        representation of the program.

ast

        as tree but with some modifications to make it identical to erlang's
        Abstract Syntax Tree.

mod

        as ast but with information about the module added so it can be
        compiled into a module.

beam

        the bytecode you get written to a file.

All this steps can be dumped to the screen (except beam that goes to a file)
using the -t option::

        # dump the lexer structure of module.fn
        fnc -t lex module.fn

        # dump the tree structure of module.fn
        fnc -t tree module.fn

        # dump the ast structure of module.fn
        fnc -t ast module.fn

        # dump the mod structure of module.fn
        fnc -t mod module.fn

        # compile the file to byte code
        fnc -t beam module.fn

        # identical to above
        fnc module.fn

An extra option is available to transform an erlang file to ast, this is used
to compare the ast generated by an erlang program with an identical program in
efene::

        fnc -t erl2ast mymodule.erl 

This option generated the same result as *-t mod* but using an erlang file as
input.
