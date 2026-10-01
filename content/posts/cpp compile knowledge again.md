+++
date = "2026-09-16"
author = "Johnny"
title = "cpp compile knowledge again"
tags = ["g++", "GNU", "Makefile"]
+++

```
CXX = g++
DEPEND = -MMD -MP
COMMAND = -o

test04: test05.o test04.o
	$(CXX) test05.o test04.o $(COMMAND) test4.exe

test05.o: test05.cpp
	$(CXX) $(DEPEND) -c test05.cpp $(COMMAND) test05.o

test04.o: test04.cpp
	$(CXX) $(DEPEND) -c test04.cpp $(COMMAND) test04.o


.PHONY: clean

clean: 
	del /Q /F *.o test04.exe

-include $(wildcard *.d)

```

---


When facing such stucture: 

test04.cpp:
```
include "test05.h"

```

test05.h:
```
func
```

test05.cpp:
```
implement the func
```

test04.cpp include test05.h, and test05.cpp implement the function defined in test05.h, we should write the makefile in the way above. Moreover, we should include test05.h in test05.cpp for safety reasons (.e.g, when we incorrectly define a function in test05.cpp, compiler will point out the unconsistency between the implementation and definition of this function) 