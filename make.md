# make
Make is a tool to compile C/C++ projects.

Install `make`:
```
sudo apt install make
```

# First simple example:
```make
# Makefile
###############
## TARGET   ##
###############
# target: prerequisites
#	commands

build: my_lib.o main.o
	g++ main.o my_lib.o -o main

main.o:
	g++ main.cc -c

my_lib.o:
	g++ my_lib.cc -c

execute:
	./main

clean:
	rm -f *.o
	rm -f main
```
In the same directory with this Makefile, you can run:
- `make build`    - to build the library and executable
- `make execute`  - to run the executable
- `make clean`    - to remove the library and executable
- `make my_lib.o` - to build only the specific library

__Note__: all TAB characters in Makefile must be tab, not space.

# Variables
The default variables in Makefile:
- `CC`: Program for compiling C programs; default cc
- `CXX`: Program for compiling C++ programs; default g++
- `CFLAGS`: Extra flags to give to the C compiler
- `CXXFLAGS`: Extra flags to give to the C++ compiler
- `CPPFLAGS`: Extra flags to give to the C preprocessor
- `LDFLAGS`: Extra flags to give to the linker

Otherwise, you can define your own variables.

To get a variable's value, we use `$`, for example: `$(CXX_STANDARD)`

__Note__: Variables can only be strings. Single or double quotes for variable names or values have no meaning to Make
```make
DEBUG = 1
EXECUTABLE_NAME = main

CXX_STANDARD = c++17
CXX_WARNINGS = -Wall -Wextra -Wpedantic
CXX = g++
CXXFLAGS = $(CXX_WARNINGS) -std=$(CXX_STANDARD)
LDFLAGS =

ifeq ($(DEBUG), 1)
CXXFLAGS += -g -O0
EXECUTABLE_NAME = mainDebug
else
CXXFLAGS += -O3
EXECUTABLE_NAME = mainRelease
endif

COMPILER_CALL = $(CXX) $(CXXFLAGS)

build: my_lib.o main.o
	$(COMPILER_CALL) main.o my_lib.o $(LDFLAGS) -o $(EXECUTABLE_NAME)

main.o:
	$(COMPILER_CALL) main.cc -c

my_lib.o:
	$(COMPILER_CALL) my_lib.cc -c

execute:
	./$(EXECUTABLE_NAME)

clean:
	rm -f *.o
	rm -f $(EXECUTABLE_NAME)
```

# Patterns
We can use patterns to do the same tasks for several source files once at a time:
- `$@`: the file name of the target
- `$<`: the name of the first ependency
- `$^`: the names of all prerequisites

Below code will scan all `.cc` source files and compile them due to the patterns defined at the end of the code, `wildcard` function and `patsubst` function
- `wildcard` - function to scan all `.cc` files
- `patsubst` - function to create `%.o` corresponding to `%.cc` in the `$(CXX_SOURCES)` directory
```make
DEBUG = 1

CXX_STANDARD = c++17
CXX_WARNINGS = -Wall -Wextra -Wpedantic
CXX = g++
CXXFLAGS = $(CXX_WARNINGS) -std=$(CXX_STANDARD)
LDFLAGS =

ifeq ($(DEBUG), 1)
CXXFLAGS += -g -O0
EXECUTABLE_NAME = mainDebug
else
CXXFLAGS += -O3
EXECUTABLE_NAME = mainRelease
endif

CXX_COMPILER_CALL = $(CXX) $(CXXFLAGS)

CXX_SOURCES = $(wildcard *.cc)
CXX_OBJECTS = $(patsubst %.cc, %.o, $(CXX_SOURCES))

##############
## TARGETS  ##
##############
build: $(CXX_OBJECTS)
	$(CXX_COMPILER_CALL) $(CXX_OBJECTS) $(LDFLAGS) -o $(EXECUTABLE_NAME)

execute:
	./$(EXECUTABLE_NAME)

clean:
	rm -f *.o
	rm -f $(EXECUTABLE_NAME)

##############
## PATTERNS ##
##############
%.o: %.cc
	$(CXX_COMPILER_CALL) -c $< -o $@
```

# Completed Makefile:
- Add variables to seperate directories: `INCLUDE_DIR`, `SOURCE_DIR`, `BUILD_DIR`
- Add variables to define warning options: `ENABLE_WARNINGS`, `WARNINGS_AS_ERRORS`
- Add `.PHONY` target (predefined) to tell make tool that which targets are not file names

More to note:
- Add `?=` to variables that user can set. If it ins't set yet, it will be the default value.
- Add `@` before any command so that it won't be printed on the terminal.
- Define `all` target to point to multiple target.
- `cd build && mkdir test` will work but if you write them seperately in two lines it won't work since each command is run in an own shell.
- More conditions keywords:
    - Check if a variable is defined: `ifdef VAR`
    - Check if a variable is empty: `ifeq ($(strip $(VAR)),)`
      
_(`strip` is to remove leading and trailing whitespace from the value of the `VAR` variable)_
```make
DEBUG ?= 1
ENABLE_WARNINGS ?= 1
WARNINGS_AS_ERRORS ?= 0

INCLUDE_DIR = include
SOURCE_DIR = src
BUILD_DIR = build

ifeq ($(ENABLE_WARNINGS), 1)
CXX_WARNINGS = -Wall -Wextra -Wpedantic
else
CXX_WARNINGS =
endif
ifeq ($(WARNINGS_AS_ERRORS), 1)
CXX_WARNINGS += -Werror
endif

CXX_STANDARD = c++17
CXX = g++
CXXFLAGS = $(CXX_WARNINGS) -std=$(CXX_STANDARD)
CPPFLAGS = -I $(INCLUDE_DIR)
LDFLAGS =

ifeq ($(DEBUG), 1)
CXXFLAGS += -g -O0
EXECUTABLE_NAME = mainDebug
else
CXXFLAGS += -O3
EXECUTABLE_NAME = mainRelease
endif

CXX_COMPILER_CALL = $(CXX) $(CXXFLAGS) $(CPPFLAGS)

CXX_SOURCES = $(wildcard $(SOURCE_DIR)/*.cc)
CXX_OBJECTS = $(patsubst $(SOURCE_DIR)/%.cc, $(BUILD_DIR)/%.o, $(CXX_SOURCES))

##############
## TARGETS  ##
##############
all: create build

create:
	@mkdir build

build: create $(CXX_OBJECTS)
	$(CXX_COMPILER_CALL) $(CXX_OBJECTS) $(LDFLAGS) -o $(BUILD_DIR)/$(EXECUTABLE_NAME)

execute:
	./$(BUILD_DIR)/$(EXECUTABLE_NAME)

clean:
	rm -f $(BUILD_DIR)/*.o
	rm -f $(BUILD_DIR)/$(EXECUTABLE_NAME)

##############
## PATTERNS ##
##############
$(BUILD_DIR)/%.o: $(SOURCE_DIR)/%.cc
	$(CXX_COMPILER_CALL) -c $< -o $@

###########
## PHONY ##
###########
.PHONY: create build execute clean
```

