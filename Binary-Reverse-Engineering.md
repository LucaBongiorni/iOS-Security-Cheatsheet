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

Now this representation is something we can wrap our brain around.  If you want to just stare at assembly: 

```
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
```

This roughly translates into ``` objc_msgSend(SetUserPrefs, setPrefs); ```

Now we have something we can really focus on -> ```[SetUserPrefs setPrefs]``` - let's use **Hopper** and navigate to that specific subroutine:

```
             -[SetUserPrefs setPrefs]:
00009d98         push       {r7, lr}                                            ; Objective C Implementation defined at 0xc150 (instance), XREF=0x40ac
00009d9a         mov        r7, sp
00009d9c         sub        sp, #0x24
00009d9e         movw       r2, #0x22a6
00009da2         movt       r2, #0x0
00009da6         add        r2, pc                                              ; @"rotlogix"
00009da8         movw       r3, #0x2b0c
00009dac         movt       r3, #0x0
00009db0         add        r3, pc                                              ; objc_ivar_offset_SetUserPrefs_username
00009db2         str        r0, [sp, #0x24 + var_20]
00009db4         str        r1, [sp, #0x24 + var_1C]
00009db6         ldr        r0, [sp, #0x24 + var_20]
00009db8         ldr        r1, [r3]                                            ; objc_ivar_offset_SetUserPrefs_username
00009dba         add        r0, r1                                              ; argument #1 for method imp___symbolstub1__objc_storeStrong
00009dbc         mov        r1, r2                                              ; argument #2 for method imp___symbolstub1__objc_storeStrong
00009dbe         blx        imp___symbolstub1__objc_storeStrong
00009dc2         movw       r0, #0x2292
00009dc6         movt       r0, #0x0
00009dca         add        r0, pc                                              ; @"rotlogix@gmail.com"
00009dcc         movw       r1, #0x2aec
00009dd0         movt       r1, #0x0
00009dd4         add        r1, pc                                              ; objc_ivar_offset_SetUserPrefs_email
00009dd6         ldr        r2, [sp, #0x24 + var_20]
00009dd8         ldr        r1, [r1]                                            ; objc_ivar_offset_SetUserPrefs_email
00009dda         add        r1, r2
00009ddc         str        r0, [sp, #0x24 + var_14]
00009dde         mov        r0, r1                                              ; argument #1 for method imp___symbolstub1__objc_storeStrong
00009de0         ldr        r1, [sp, #0x24 + var_14]                            ; argument #2 for method imp___symbolstub1__objc_storeStrong
00009de2         blx        imp___symbolstub1__objc_storeStrong
00009de6         movw       r0, #0x2256
00009dea         movt       r0, #0x0
00009dee         add        r0, pc                                              ; imp___nl_symbol_ptr__objc_msgSend
00009df0         ldr        r0, [r0]                                            ; imp___nl_symbol_ptr__objc_msgSend
00009df2         movw       r1, #0x2a22
00009df6         movt       r1, #0x0
00009dfa         add        r1, pc                                              ; @selector(standardUserDefaults)
00009dfc         movw       r2, #0x2a8c
00009e00         movt       r2, #0x0
00009e04         add        r2, pc                                              ; objc_cls_ref_NSUserDefaults
00009e06         ldr        r2, [r2]                                            ; objc_cls_ref_NSUserDefaults
00009e08         ldr        r1, [r1]                                            ; @selector(standardUserDefaults)
00009e0a         str        r0, [sp, #0x24 + var_10]
00009e0c         mov        r0, r2
00009e0e         ldr        r2, [sp, #0x24 + var_10]
00009e10         blx        r2                                                  ; _OBJC_CLASS_$_NSUserDefaults
00009e12         mov        r7, r7
00009e14         blx        imp___symbolstub1__objc_retainAutoreleasedReturnValue
00009e18         movw       r1, #0x224c
00009e1c         movt       r1, #0x0
00009e20         add        r1, pc                                              ; @"username"
00009e22         movw       r2, #0x221a
00009e26         movt       r2, #0x0
00009e2a         add        r2, pc                                              ; imp___nl_symbol_ptr__objc_msgSend
00009e2c         ldr        r2, [r2]                                            ; imp___nl_symbol_ptr__objc_msgSend
00009e2e         movw       r3, #0x29ea
00009e32         movt       r3, #0x0
00009e36         add        r3, pc                                              ; @selector(setValue:forKey:)
00009e38         movw       sb, #0x2a7c
00009e3c         movt       sb, #0x0
00009e40         add        sb, pc                                              ; objc_ivar_offset_SetUserPrefs_username
00009e42         str        r0, [sp, #0x24 + var_18]
00009e44         ldr        r0, [sp, #0x24 + var_18]
00009e46         ldr.w      ip, [sp, #0x24 + var_20]
00009e4a         ldr.w      sb, [sb]                                            ; objc_ivar_offset_SetUserPrefs_username
00009e4e         add        sb, ip
00009e50         ldr.w      sb, [sb]
00009e54         ldr        r3, [r3]                                            ; @selector(setValue:forKey:)
00009e56         str        r1, [sp, #0x24 + var_C]
00009e58         mov        r1, r3
00009e5a         str        r2, [sp, #0x24 + var_8]
00009e5c         mov        r2, sb
00009e5e         ldr        r3, [sp, #0x24 + var_C]
00009e60         ldr.w      sb, [sp, #0x24 + var_8]
00009e64         blx        sb
00009e66         movw       r0, #0x220e
00009e6a         movt       r0, #0x0
00009e6e         add        r0, pc                                              ; @"email"
00009e70         movw       r1, #0x21cc
00009e74         movt       r1, #0x0
00009e78         add        r1, pc                                              ; imp___nl_symbol_ptr__objc_msgSend
00009e7a         ldr        r1, [r1]                                            ; imp___nl_symbol_ptr__objc_msgSend
00009e7c         movw       r2, #0x299c
00009e80         movt       r2, #0x0
00009e84         add        r2, pc                                              ; @selector(setValue:forKey:)
00009e86         movw       r3, #0x2a32
00009e8a         movt       r3, #0x0
00009e8e         add        r3, pc                                              ; objc_ivar_offset_SetUserPrefs_email
00009e90         ldr.w      sb, [sp, #0x24 + var_18]
00009e94         ldr.w      ip, [sp, #0x24 + var_20]
00009e98         ldr        r3, [r3]                                            ; objc_ivar_offset_SetUserPrefs_email
00009e9a         add        r3, ip
00009e9c         ldr        r3, [r3]
00009e9e         ldr        r2, [r2]                                            ; @selector(setValue:forKey:)
00009ea0         str        r0, [sp, #0x24 + var_4]
00009ea2         mov        r0, sb
00009ea4         str        r1, [sp, #0x24 + var_0]
00009ea6         mov        r1, r2
00009ea8         mov        r2, r3
00009eaa         ldr        r3, [sp, #0x24 + var_4]
00009eac         ldr.w      sb, [sp, #0x24 + var_0]
00009eb0         blx        sb                                                  ; @"email"
00009eb2         movs       r1, #0x0                                            ; argument #2 for method imp___symbolstub1__objc_storeStrong
00009eb4         add        r0, sp, #0x18                                       ; argument #1 for method imp___symbolstub1__objc_storeStrong
00009eb6         blx        imp___symbolstub1__objc_storeStrong
00009eba         add        sp, #0x24
00009ebc         pop        {r7, pc}
                        ; endp
```

Because of the message passing conversion process -> this generate quite a few instructions.
