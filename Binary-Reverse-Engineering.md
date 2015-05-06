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

```
-[AppDelegate application:didFinishLaunchingWithOptions:]:
00009f04         push       {r7, lr}                                            ; Objective C Implementation defined at 0xc63c (instance)
00009f06         mov        r7, sp
00009f08         sub        sp, #0x28
00009f0a         add.w      sb, sp, #0x1c
00009f0e         movw       ip, #0x0
00009f12         str        r0, [sp, #0x28 + var_24]
00009f14         str        r1, [sp, #0x28 + var_20]
00009f16         str.w      ip, [sp, #0x28 + var_1C]
00009f1a         mov        r0, sb                                              ; argument #1 for method imp___symbolstub1__objc_storeStrong
00009f1c         mov        r1, r2                                              ; argument #2 for method imp___symbolstub1__objc_storeStrong
00009f1e         str        r3, [sp, #0x28 + var_C]
00009f20         blx        imp___symbolstub1__objc_storeStrong
00009f24         add        r0, sp, #0x18                                       ; argument #1 for method imp___symbolstub1__objc_storeStrong
00009f26         movs       r1, #0x0
00009f28         str        r1, [sp, #0x28 + var_18]
00009f2a         ldr        r1, [sp, #0x28 + var_C]                             ; argument #2 for method imp___symbolstub1__objc_storeStrong
00009f2c         blx        imp___symbolstub1__objc_storeStrong
00009f30         movw       r0, #0x210c
00009f34         movt       r0, #0x0
00009f38         add        r0, pc                                              ; imp___nl_symbol_ptr__objc_msgSend
00009f3a         ldr        r0, [r0]                                            ; imp___nl_symbol_ptr__objc_msgSend
00009f3c         movw       r1, #0x28e0
00009f40         movt       r1, #0x0
00009f44         add        r1, pc                                              ; @selector(alloc)
00009f46         movw       r2, #0x2946
00009f4a         movt       r2, #0x0
00009f4e         add        r2, pc                                              ; objc_cls_ref_SetUserPrefs
00009f50         ldr        r2, [r2]                                            ; objc_cls_ref_SetUserPrefs
00009f52         ldr        r1, [r1]                                            ; @selector(alloc)
00009f54         str        r0, [sp, #0x28 + var_8]
00009f56         mov        r0, r2
00009f58         ldr        r2, [sp, #0x28 + var_8]
00009f5a         blx        r2                                                  ; objc_class_SetUserPrefs
00009f5c         movw       r1, #0x20e0
00009f60         movt       r1, #0x0
00009f64         add        r1, pc                                              ; imp___nl_symbol_ptr__objc_msgSend
00009f66         ldr        r1, [r1]                                            ; imp___nl_symbol_ptr__objc_msgSend
00009f68         movw       r2, #0x28b8
00009f6c         movt       r2, #0x0
00009f70         add        r2, pc                                              ; @selector(init)
00009f72         ldr        r2, [r2]                                            ; @selector(init)
00009f74         str        r1, [sp, #0x28 + var_4]
00009f76         mov        r1, r2
00009f78         ldr        r2, [sp, #0x28 + var_4]
00009f7a         blx        r2                                                  ; 0xae00
00009f7c         movw       r1, #0x20c0
00009f80         movt       r1, #0x0
00009f84         add        r1, pc                                              ; imp___nl_symbol_ptr__objc_msgSend
00009f86         ldr        r1, [r1]                                            ; imp___nl_symbol_ptr__objc_msgSend
00009f88         movw       r2, #0x289c
00009f8c         movt       r2, #0x0
00009f90         add        r2, pc                                              ; @selector(setPrefs)
00009f92         str        r0, [sp, #0x28 + var_14]
00009f94         ldr        r0, [sp, #0x28 + var_14]
00009f96         ldr        r2, [r2]                                            ; @selector(setPrefs)
00009f98         str        r1, [sp, #0x28 + var_0]
00009f9a         mov        r1, r2
00009f9c         ldr        r2, [sp, #0x28 + var_0]
00009f9e         blx        r2                                                  ; 0xadd4
00009fa0         movs       r1, #0x0                                            ; argument #2 for method imp___symbolstub1__objc_storeStrong
00009fa2         add        r0, sp, #0x14                                       ; argument #1 for method imp___symbolstub1__objc_storeStrong
00009fa4         movs       r2, #0x1
00009fa6         str        r2, [sp, #0x28 + var_10]
00009fa8         blx        imp___symbolstub1__objc_storeStrong
00009fac         movs       r1, #0x0                                            ; argument #2 for method imp___symbolstub1__objc_storeStrong
00009fae         add        r0, sp, #0x18                                       ; argument #1 for method imp___symbolstub1__objc_storeStrong
00009fb0         blx        imp___symbolstub1__objc_storeStrong
00009fb4         movs       r1, #0x0                                            ; argument #2 for method imp___symbolstub1__objc_storeStrong
00009fb6         add        r0, sp, #0x1c                                       ; argument #1 for method imp___symbolstub1__objc_storeStrong
00009fb8         blx        imp___symbolstub1__objc_storeStrong
00009fbc         movs       r0, #0x1
00009fbe         sxtb       r0, r0
00009fc0         add        sp, #0x28
00009fc2         pop        {r7, pc}
                        ; endp
```

Breaking down the ARM assembly can be a bit daunting, especially since static diassembler have a hard time representing the dynamic dispatching nature of Objective-C.  So how do we simplify this for ourselves?

First we should already understand the messaging mechanics of objc_msgSend() and what the syntax will be if you were to create a new instance object in Objective-C. 

Flow:

  - alloc
  - init
  - methodToCall
  - release

The compiler will convert all of these into ```objc(ID ReceiverObject, SEL method, args .. );``` respectively.  We also have to remember that when looking at this assembly, the runtime is will check the ```isa``` pointer of the instance object itself.  This points directly at the class structure, and is also used to look up the corresponding ```SEL``` method in a dispatch table.

**Hopper** offers pretty solid functionality that will attempt to convert this into sudo code: 

```
char -[AppDelegate application:didFinishLaunchingWithOptions:](void * self, void * _cmd, void * arg2, void * arg3) {
    var_24 = self;
    var_20 = _cmd;
    var_1C = 0x0;
    var_C = arg3;
    objc_storeStrong((sp - 0x28) + 0x1c, arg2);
    var_18 = 0x0;
    objc_storeStrong((sp - 0x28) + 0x18, var_C);
    r0 = *objc_msgSend;
    var_8 = r0;
    objc_class_SetUserPrefs(SetUserPrefs, @selector(alloc), var_8);
    r1 = *objc_msgSend;
    var_4 = r1;
    r0 = loc_ae00();
    r1 = *objc_msgSend;
    var_14 = r0;
    var_0 = r1;
    loc_add4(var_14, @selector(setPrefs), var_0);
    var_10 = 0x1;
    objc_storeStrong((sp - 0x28) + 0x14, 0x0);
    objc_storeStrong((sp - 0x28) + 0x18, 0x0);
    objc_storeStrong((sp - 0x28) + 0x1c, 0x0);
    return r0;
}
```


