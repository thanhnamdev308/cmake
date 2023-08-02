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
## SYNTAX   ##
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
DEBUG = True
EXECUTABLE_NAME = main

CXX_STANDARD = c++17
CXX_WARNINGS = -Wall -Wextra -Wpedantic
CXX = g++
CXXFLAGS = $(CXX_WARNINGS) -std=$(CXX_STANDARD)
LDFLAGS =

ifeq ($(DEBUG), True)
CXXFLAGS += -g -O0
else
CXXFLAGS += -O3
endif

COMPILER_CALL = $(CXX) $(CXXFLAGS)

build: my_lib.o main.o
	$(COMPILER_CALL) main.o my_lib.o $(LDFLAGS) -o $(EXECUTABLE_NAME)

main.o:
	$(COMPILER_CALL) main.cc -c

my_lib.o:
	$(COMPILER_CALL) my_lib.cc -c

execute:
	./main

clean:
	rm -f *.o
	rm -f main
```

