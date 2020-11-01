Hopefully this is a useful document for getting started using the language in its curent state.
Additions, improvements, and suggestions welcome! :)

Last updated 2020-11-01
Written 2020-10-31



# Beta info

The preferred channel for reporting bugs / feedback is via the "language" email address,
so things don't get missed.

Feel free to ask for help / ask questions in the beta chat room!



# Getting started

Things to read:
* README.txt (obviously)
* how_to (obviously)
* Read module names to get a sense of what's available
* List of community-made libraries and tools: https://github.com/smari/awesome-jai

Searching for stuff (function names, module names, etc.) in the provided code
(modules, examples, and how_to) is a common flow - to find example code,
and generally figure out how things work.
Grep variant tools are useful for this, such as:
ripgrep - https://github.com/BurntSushi/ripgrep
BareGrep - https://www.baremetalsoft.com/baregrep

To discover functionality, the main flow is to browse the source code of
modules and read their comments and function names. (And function bodies if you want!)
The exmples/ folder found in some modules can be helpful.
Modules I especially recommend looking at:
Basic
String - additionally, reading these function bodies is educational for how to manipulate strings.
Hash_Table
Math
File
Compiler

A directory structure that some people use is:
X:/
    jai/            - the distributed compiler folder
        bin/
        modules/
        ...
    jai_modules/    - a folder for your own modules or downloaded modules (the compiler need to be configured to look here, using modules_search_path_array, explained later)
    my_project_1/
    my_project_2/
    ...
"X:/" could be whatever directory you want. For example, mine is C:/Patrick/Projects/jai/.
To update the compiler, just replace the jai/ folder.



# General

This section is intended to be read after the how_tos.
These are things that are not yet explicitly covered in the how_tos or readme
as of the time of this writing:

Debuggers:
You can use a debugger with an exe and the generated .pdb file.
For example, in Visual Studio, open the .exe file and you can set breakpoints and run.

Crashes and the Debug module:
If your program crashes, as of the time of this writing, the program will stop
and nothing will be printed out.
You can use the Debug module to print the stack trace on crashes:
#import "Basic";
Debug :: #import "Debug";
main :: () {
    Debug.init();
    free(cast(*void) 10000000000); // Attempt to cause a crash
    print("done\n");
}

Resizable arrays:
Functions for manipulating resizable arrays are in modules/Basic/Array.jai.
The definition for Resizable_Array is in modules/Preload.jai, but you don't really need to read it.
Example usage:
array: [..]int;
array_add(*array, 4);
array_add(*array, 6);
array_insert_at(*array, 8, 1);
array[2] = 10;
array_reset(*array);

Pointer operators:
variable_a: int;
variable_b: *int;
variable_b = *variable_a;  // address-of operator (not official name)
variable_a = <<variable_b; // dereference operator (not official name)

Memory management basics:
Search the modules/ and examples/ folders for example usage.
These are all defined in modules/Basic/module.jai:
New         Allocate and initialize memory for the given type. e.g. to allocate a new struct, use this.
NewArray    Allocate and initialize memory for an array of the given type.
alloc       Allocate memory
free        Free memory

Compiler directives:
This list is very rough, may miss some important ones, and descriptions are just rudimentary.
The ones with descriptions are the ones I use / see used most commonly.
Search the modules/ and examples/ folders for example usage.
#add_context        Add a declaration to the context
#align
#assert
#bake
#c_call
#caller_location
#char               Character literal
#code
#compiler
#complete           Ensure an if-case satement checks all values of the enum
#define
#else
#endif
#expand
#file
#filepath           The directory of the running executable
#foreign
#foreign_library
#foreign_system_library
#if
#ifdef
#ifndef
#import             Import a module
#insert
#insert_internal
#intrinsic
#load               Load source code, as if it were placed right here (very commonly used)
#modify
#module_parameters
#must               Require a return value to be assigned to a variable
#no_abc
#no_alias
#no_padding
#place
#placeholder
#program_export
#run                Run this code right now (compile time)
#run_and_insert
#runtime_support
#scope_export       Set the scope for future declarations to be accessible by code that imports this module
#scope_file         Set the scope for foture declarations to just this file
#scope_module       Set the scope for foture declarations to just this module
#specified          Declare intention of maintaining enum values compatibility over time
#string             String literal with delimiter
#symmetric
#through
#type               Type literal, e.g. for function types
#type_info_none
#type_info_procedures_are_void_pointers



# Useful phrases

Example compile_me.jai:
Based on modules/Compiler/examples/compile_me.jai
and examples/snake/first.jai

Compiler :: #import "Compiler";

#run build();

build :: () {
    opts := Compiler.get_build_options();
    opts.output_type = .NO_OUTPUT;
    Compiler.set_build_options(opts);

    opts.output_type = .EXECUTABLE;
    opts.output_executable_name = "executable_name";
    array_add(*opts.modules_search_path_array, "../jai_modules"); // This assumes folders are laid out as mentioned earlier
    // See modules/Compiler for more build options

    ws := Compiler.compiler_create_workspace("");
    Compiler.set_build_options(opts, ws);
    Compiler.add_build_file("main.jai", ws);
    // main.jai should contain a function named "main".

    // If "run" is supplied as a compiler command line argument, like
    //   jai compile_me.jai -- run
    // then run the game immediately, as a convenience.
    args := Compiler.compiler_get_command_line_arguments();
    for args {
        if (it == "run") {
            Compiler.add_build_string("#run main();", ws);
            break;
        }
    }
}

Struct "using base" syntax:
This is also described in the types how_to.

Entity :: struct {
    type: Type; // A Type variable is a convenient way to keep track of what "sub-type" entities are. This could also be an enum.
    x: float;
    y: float;
}

Snake :: struct {
    // This makes the Snake struct contain an Entity.
    // It is convenient to make this the first thing in the struct, so that we can treat a pointer to this as an Entity.
    // The "using" keyword is a convenience to let us access things as snake.x instead of snake.base.x.
    using base: Entity;
    snake_color: int;
}

Player :: struct {
    using base: Entity;
    player_index: int;
}

entities: [..]*Entity;

cool_snake := New(Snake);
cool_snake.type = Snake;
cool_snake.x = 2;
cool_snake.y = 4;
cool_snake.snake_color = 1234;
array_add(*entities, cool_snake);

for entities {
    if it.x > 1 {
        if it.type == Snake {
            snake := cast(*Snake) it;
            print("%\n", snake.snake_color);
        }
        else if it.type == Player {
            // ...
        }
    }
}
free(cool_snake);

Anonymous struct variable, a useful grouping for data that only has one copy:
state: struct {
    variable_a: int;
    variable_b: int;
};
some_function :: () {
    state.variable_a = 4;
}

Anonymous enum variable, useful for an enum that only has one copy:
state: enum {
    MEMBER_A;
    MEMBER_B;
};
some_function :: () {
    state = .MEMBER_B;
}

--- syntax means the variable will be uniitialized. As in:
variable_c: Vector3 = ---;
