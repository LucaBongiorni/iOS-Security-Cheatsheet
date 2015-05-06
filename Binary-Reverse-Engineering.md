Binary Reverse Engineering
==========================

Hopper
------

**1.) Loading the Binary**

```File -> Read Executable to Disassemble -> FAT Binary -> Select Architecture```

I like to start reversing at ``` -[AppDelegate applicationdidFinishLaunchingWithOptions ``` - and here is why: 

  - The AppDelegate is the root object of the application
  - Implements methods are defined in the ```UIApplication``` object
  - Handles start up events
