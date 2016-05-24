---
layout: post
title:  Style Guide
author: Elsa Gonsiorowski
category: ross-dev
---

ROSS doesn't officially enforce any particular style guide.
Over the years, the different contributers have each had their own conventions.
This post will document the conventions currently in place and outline a style guide for future contributions.

## Current Style Guide

### Whitespace

ROSS codes uses the Unix-style line endings.
Additionally, it is best practice to removed trailing whitespace from lines in the code.
Each file should end with a blank line (the final character should be a `\n`).

### Variable Names

ROSS prefers the use of underscores (`_`) between words in variable names (rather than CamelCase).

### Naming Convection

The `tw` prefix is used throughout ROSS: in variable names, function names, and file names.
This stands for *Time Warp* and denotes variables/functions/files that are essential to the time warp kernel.

Filenames that start with the `ross` prefix denote parts of the code that are particular to the ROSS simulator.
At one point in time there was a difference between what constitutes *ROSS* and what constitutes *Time Warp*, but today this difference insignificant.

Some variable start with a `g_` prefix.
This denotes a global variable.

### Doxygen In-Line Comments

The ROSS core code base is parsed by Doxygen to generate documentation.

For comment blocks, ROSS uses the JavaDoc style, which starts the block with `/**`.
Example:

```
    /**
     * @file buddy.h
     * @brief Buddy-system memory allocator
     */
```

Convention is that commands begin with an `@` symbol (rather than a `\`).
The Doxygen commands used in the ROSS code include (details of these commands can be found [here](http://www.stack.nl/~dimitri/doxygen/manual/commands.html)):

- `@file`
- `@brief`
- `@param`
- `@return`
- `@attention`
- `@bug`
- `@section`
- `@mainpage`

For in-line comments for struct member variables, ROSS uses the `/**<` notation.
Example:

```
    struct tw_lptype {
        init_f init; /**< @brief LP setup routine */
        pre_run_f pre_run; /**< @brief Second stage LP initialization */
    };
```

More details on Doxygen code-commenting style can be found [here](http://www.stack.nl/~dimitri/doxygen/manual/docblocks.html).
