+++
title = "Meson Build System Quick Start"
description = "A fast and practical guide for setting up C projects with Meson"
date = "2024-11-01"
template = "post.html"
+++

In this article, I'll walk you through a quick way to dive into the Meson Build System, starting from scratch to create an executable project, add a library, and more.

Let’s get started!

### What Will We Cover?
1. Creating a basic project structure
2. Setting up and configuring your project files
3. Adding and organizing source and header files

#### Step 1: Create a New Project

To start, we’ll create a new project that compiles into an executable and is installable on your system.

Use this [LINK](https://mesonbuild.com/SimpleStart.html#installing-meson) to install Meson if you haven't already.

```sh,linenos
mkdir myexec
meson init --type executable -l c -n myexec --builddir build -C ./myexec
```

Here’s what each part of the command does:
- `init`: Initializes a new project.
- `--type`: Defines the project type; `executable` for executables and `library` for libraries.
- `-l`: Specifies the project language (e.g., `c`, `c++`, `rust`, etc.).
- `-n`: Sets the project name.
- `--builddir`: Names the build directory.
- `-C`: Changes to the specified folder and initializes the project there.

#### Step 2: Configure the Project

Navigate into the `myexec` folder to set up the file structure.

```sh,linenos
cd myexec
mkdir src include build
mv myexec.c src/main.c
```

Next, create some additional source and header files.

```sh,linenos
touch src/funcs.c
touch include/funcs.h
```

Here’s the content for `funcs.h`:

```c,linenos
#pragma once

int call_me_daddy();
```

And for `funcs.c`:

```c,linenos
#include "funcs.h"
#include <stdio.h>

int call_me_daddy() {
    printf("Hi Daddy!!!");
    return 0;    
}
```

#### Step 3: Update the `meson.build` File

Open the `meson.build` file and update it to look like this:

```c,linenos
project(
    'myexec',
    'c',
    version : '0.0.0',
    default_options : [
        'c_std=c2x',
        'warning_level=3',
    ],
    meson_version : '>=1.5.0'
)

build_args = [
    '-DNAME=' + meson.project_name(),
    '-DVERSION=' + meson.project_version(),
]

header = include_directories('include')

myexec = executable(
    meson.project_name(),
    sources : run_command('find', 'src', '-depth', '-type', 'f', check : true).stdout().strip().split('\n'),
    install : true,
    native : true,
    c_args : build_args,
    include_directories : header
)
```

Check these references:
- [executable()](https://mesonbuild.com/Reference-manual_functions.html#executable)
- [project()](https://mesonbuild.com/Reference-manual_functions.html#project)
- [include_directories()](https://mesonbuild.com/Reference-manual_functions.html#include_directories)
- [run_command()](https://mesonbuild.com/Reference-manual_functions.html#run_command)

#### Step 4: Add a Test Directory

Before proceeding, let’s create a test directory:

```sh,linenos
mkdir test
touch test/test.c
```

#### Step 5: Set Up Library and Test Executable

Finally, add a library and test executable to your `meson.build` file:

```c,linenos
myexec_lib = static_library(
    meson.project_name() + '_lib',
    sources : run_command('find', 'src', '-depth', '-type', 'f', '-not', '-name', 'main.c', check : true).stdout().strip().split('\n'),
    c_args : build_args,
    include_directories : header
)

myexec_test = executable(
    meson.project_name() + '_test',
    sources : run_command('find', 'test', '-depth', '-type', 'f', check : true).stdout().strip().split('\n'),
    c_args : build_args,
    dependencies : [dep]
)

test('test', myexec_test)
```

You can use `shared_library` instead.

- [static_library()](https://mesonbuild.com/Reference-manual_functions.html#static_library)
- [shared_library()](https://mesonbuild.com/Reference-manual_functions.html#shared_library)
- [test()](https://mesonbuild.com/Reference-manual_functions.html#test)

If you have a lot of test files that execute separately, use this instead of `myexec_test` and `test`:

```c,linenos
foreach test_file : run_command('find', 'test', '-depth', '-type', 'f', check : true).stdout().strip().split('\n')
    test_name = test_file.split('/')[-1].substring(0, -2)
    test_exe = executable(
        test_name,
        test_file,
        link_with : myexec_lib,
        build_by_default : false,
        d_unittest : true,
        include_directories : src_headers
    )
    test(test_name, test_exe)
endforeach
```

Then add test files:

```sh,linenos
touch test/test_call_me_daddy.c
```

And the content is this:

```c,linenos
#include "funcs.h"

int main() {
    int res = call_me_daddy();
    assert(res == 0);
}
```

#### Step 6: Setup, Compile, Test, Run

- Setup:
```sh,linenos
meson setup build
```

- Compile:
```sh,linenos
meson compile -v -C build
```

- Test:
```sh,linenos
meson test -v -C build
```

- Run:
```sh,linenos
./build/myexec
```

And that's it! I hope you enjoy it. If you have any questions, feel free to reach out:
- [Telegram](https://t.me/my_acrp_exp)
- [X (formerly Twitter)](https://x.com/0_keeper_1)