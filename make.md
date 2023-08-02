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

# Variable

