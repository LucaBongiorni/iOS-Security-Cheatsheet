Binary Enumeration
------------------

First we need to figure out if we are dealing with a Universal 'FAT' binary, or Macho-0: 

```
[~/Development/python/mach-O-parser]> file NSUserDefaultsExample
NSUserDefaultsExample: Mach-O universal binary with 2 architectures
NSUserDefaultsExample (for architecture armv7):	Mach-O executable arm
NSUserDefaultsExample (for architecture arm64):	Mach-O 64-bit executable
```
The file command reveals that the target binary is a 'FAT' one, so now we can use otool to inspect the 'FAT' headers of the binary.  A FAT binary wraps up two binaries of different architectures, which iOS will inspect throught the FAT headers, and load into memory the appropriate one:

```
[~/Development/python/mach-O-parser]> otool -fV NSUserDefaultsExample
Fat headers
fat_magic FAT_MAGIC
nfat_arch 2
architecture armv7
    cputype CPU_TYPE_ARM
    cpusubtype CPU_SUBTYPE_ARM_V7
    capabilities 0x0
    offset 16384
    size 62480
    align 2^14 (16384)
architecture arm64
    cputype CPU_TYPE_ARM64
    cpusubtype CPU_SUBTYPE_ARM64_ALL
    capabilities 0x0
    offset 81920
    size 62624
    align 2^14 (16384)
```

We can see the architectures listed: 

  1. arm7
  2. arm64

**Binary Protections**

We can easily detect whether each binary has been compiled with PIE (ASLR): 

```
[~/Development/python/mach-O-parser]> otool -hV NSUserDefaultsExample
NSUserDefaultsExample (architecture armv7):
Mach header
      magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
   MH_MAGIC     ARM         V7  0x00     EXECUTE    24       2404   NOUNDEFS DYLDLINK TWOLEVEL PIE
NSUserDefaultsExample (architecture arm64):
Mach header
      magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64   ARM64        ALL  0x00     EXECUTE    24       2816   NOUNDEFS DYLDLINK TWOLEVEL PIE
```

PIE is included within the flags for each binary, so now we know we are dealing with binaries compiled with ASLR.  

If the application came from the Apple Store, then we know it is probably encrypted.  We can use otool to read the cryptid field within the LC_SEGMENT:

```
[~/Development/python/mach-O-parser]> otool -lV NSUserDefaultsExample | grep 'cryptid'
      cryptid 0
      cryptid 0
```

Our binaries are not encrypted, because they are ad-hoc distributions from XCode and not prepared for store deployment.

**Dumping the Objective-C Segment**

This can be easily accomplished with ```otool```:

```
[~/Development/python/mach-O-parser]> otool -o NSUserDefaultsExample
NSUserDefaultsExample (architecture armv7):
Contents of (__DATA,__objc_classlist) section
0000c100 0xc8ec
           isa 0xc8d8
    superclass 0x0
         cache 0x0
        vtable 0x0
          data 0xc198 (struct class_ro_t *)
                    flags 0x184 RO_HAS_CXX_STRUCTORS
            instanceStart 4
             instanceSize 12
               ivarLayout 0xb8bb
                layout map: 0x02
                     name 0xb8ae SetUserPrefs
              baseMethods 0xc148 (struct method_list_t *)
		   entsize 12
		     count 2
		      name 0xadd5 setPrefs
		     types 0xb8f9 v8@0:4
		       imp 0x9d99
		      name 0xadde .cxx_destruct
		     types 0xb8f9 v8@0:4
		       imp 0x9ec1
            baseProtocols 0x0
                    ivars 0xc168
                    entsize 20
                      count 2
			   offset 0xc8c0 4
			     name 0xadec username
			     type 0xb900 @"NSString"
			alignment 2
			     size 4
			   offset 0xc8c4 8
			     name 0xadf5 email
			     type 0xb900 @"NSString"
			alignment 2
			     size 4
           weakIvarLayout 0x0
           baseProperties 0x0
Meta Class
           isa 0x0
    superclass 0x0
         cache 0x0
        vtable 0x0
          data 0xc120 (struct class_ro_t *)
                    flags 0x185 RO_META RO_HAS_CXX_STRUCTORS
            instanceStart 20
             instanceSize 20
               ivarLayout 0x0
                     name 0xb8ae SetUserPrefs
              baseMethods 0x0 (struct method_list_t *)
            baseProtocols 0x0
                    ivars 0x0
           weakIvarLayout 0x0
           baseProperties 0x0
0000c104 0xc914
           isa 0xc900
    superclass 0x0
         cache 0x0
        vtable 0x0
          data 0xc788 (struct class_ro_t *)
                    flags 0x184 RO_HAS_CXX_STRUCTORS
            instanceStart 4
             instanceSize 20
               ivarLayout 0xb8e8
                layout map: 0x04
                     name 0xb8bd AppDelegate
              baseMethods 0xc634 (struct method_list_t *)
		   entsize 12
		     count 14
		      name 0xb029 application:didFinishLaunchingWithOptions:
		     types 0xb92f c16@0:4@8@12
		       imp 0x9f05
		      name 0xb070 applicationWillResignActive:
		     types 0xb90c v12@0:4@8
		       imp 0x9fc5
		      name 0xb469 applicationDidEnterBackground:
		     types 0xb90c v12@0:4@8
		       imp 0x9fed
		      name 0xb488 applicationWillEnterForeground:
		     types 0xb90c v12@0:4@8
		       imp 0xa015
		      name 0xb054 applicationDidBecomeActive:
		     types 0xb90c v12@0:4@8
		       imp 0xa03d
		      name 0xb0fe applicationWillTerminate:
		     types 0xb90c v12@0:4@8
		       imp 0xa065
		      name 0xaeb7 applicationDocumentsDirectory
		     types 0xbead @8@0:4
		       imp 0xa0b1
		      name 0xae88 managedObjectModel
		     types 0xbead @8@0:4
		       imp 0xa12d
		      name 0xaf7e persistentStoreCoordinator
		     types 0xbead @8@0:4
		       imp 0xa295
		      name 0xafb8 managedObjectContext
		     types 0xbead @8@0:4
		       imp 0xa679
		      name 0xae06 saveContext
		     types 0xb8f9 v8@0:4
		       imp 0xa7bd
		      name 0xadde .cxx_destruct
		     types 0xb8f9 v8@0:4
		       imp 0xa909
		      name 0xb719 window
		     types 0xbead @8@0:4
		       imp 0xa8c1
		      name 0xb720 setWindow:
		     types 0xb90c v12@0:4@8
		       imp 0xa8dd
            baseProtocols 0xc600
                      count 1
		      list[0] 0xc97c (struct protocol_t *)
			      isa 0x0
			     name 0xb8c9 UIApplicationDelegate
			protocols 0xc338
		  instanceMethods 0x0 (struct method_list_t *)
		     classMethods 0x0 (struct method_list_t *)
	  optionalInstanceMethods 0xc344
	     optionalClassMethods 0x0
	       instanceProperties 0xc548
                    ivars 0xc6e4
                    entsize 20
                      count 4
			   offset 0xc8d0 4
			     name 0xb83c _managedObjectContext
			     type 0xbf71 @"NSManagedObjectContext"
			alignment 2
			     size 4
			   offset 0xc8c8 8
			     name 0xb852 _managedObjectModel
			     type 0xbf8b @"NSManagedObjectModel"
			alignment 2
			     size 4
			   offset 0xc8cc 12
			     name 0xb866 _persistentStoreCoordinator
			     type 0xbfa3 @"NSPersistentStoreCoordinator"
			alignment 2
			     size 4
			   offset 0xc8d4 16
			     name 0xb882 _window
			     type 0xbfc3 @"UIWindow"
			alignment 2
			     size 4
           weakIvarLayout 0x0
           baseProperties 0xc740
                    entsize 8
                      count 8
			     name 0xac90 window
			attributes xaca8 T@"UIWindow",&,N,V_window
			     name 0xacc2 managedObjectContext
			attributes xacd7 T@"NSManagedObjectContext",R,N,V_managedObjectContext
			     name 0xad0d managedObjectModel
			attributes xad20 T@"NSManagedObjectModel",R,N,V_managedObjectModel
			     name 0xad52 persistentStoreCoordinator
			attributes xad6d T@"NSPersistentStoreCoordinator",R,N,V_persistentStoreCoordinator
			     name 0xac48 hash
			attributes xac4d TI,R
			     name 0xac52 superclass
			attributes xac5d T#,R
			     name 0xac62 description
			attributes xac6e T@"NSString",R,C
			     name 0xac7f debugDescription
			attributes xac6e T@"NSString",R,C
Meta Class
           isa 0x0
    superclass 0x0
         cache 0x0
        vtable 0x0
          data 0xc60c (struct class_ro_t *)
                    flags 0x185 RO_META RO_HAS_CXX_STRUCTORS
            instanceStart 20
             instanceSize 20
               ivarLayout 0x0
                     name 0xb8bd AppDelegate
              baseMethods 0x0 (struct method_list_t *)
            baseProtocols 0xc600
                      count 1
		      list[0] 0xc97c (struct protocol_t *)
			      isa 0x0
			     name 0xb8c9 UIApplicationDelegate
			protocols 0xc338
		  instanceMethods 0x0 (struct method_list_t *)
		     classMethods 0x0 (struct method_list_t *)
	  optionalInstanceMethods 0xc344
	     optionalClassMethods 0x0
	       instanceProperties 0xc548
                    ivars 0x0
           weakIvarLayout 0x0
           baseProperties 0x0
0000c108 0xc928
           isa 0xc93c
    superclass 0x0
         cache 0x0
        vtable 0x0
          data 0xc7f8 (struct class_ro_t *)
                    flags 0x80
            instanceStart 162
             instanceSize 162
               ivarLayout 0x0
                     name 0xb8ea ViewController
              baseMethods 0xc7d8 (struct method_list_t *)
		   entsize 12
		     count 2
		      name 0xb88a viewDidLoad
		     types 0xb8f9 v8@0:4
		       imp 0xa9fd
		      name 0xb896 didReceiveMemoryWarning
		     types 0xb8f9 v8@0:4
		       imp 0xaa41
            baseProtocols 0x0
                    ivars 0x0
           weakIvarLayout 0x0
           baseProperties 0x0
Meta Class
           isa 0x0
    superclass 0x0
         cache 0x0
        vtable 0x0
          data 0xc7b0 (struct class_ro_t *)
                    flags 0x81 RO_META
            instanceStart 20
             instanceSize 20
               ivarLayout 0x0
                     name 0xb8ea ViewController
              baseMethods 0x0 (struct method_list_t *)
            baseProtocols 0x0
                    ivars 0x0
           weakIvarLayout 0x0
           baseProperties 0x0
Contents of (__DATA,__objc_classrefs) section
0000c894 0x0
0000c898 0xc8ec
0000c89c 0x0
0000c8a0 0x0
0000c8a4 0x0
0000c8a8 0x0
0000c8ac 0x0
0000c8b0 0x0
0000c8b4 0x0
0000c8b8 0xc914
Contents of (__DATA,__objc_superrefs) section
0000c8bc 0xc928
Contents of (__DATA,__objc_protolist) section
0000c10c 0xc950
0000c110 0xc97c
Contents of (__DATA,__objc_imageinfo) section
  version 0
    flags 0x0
NSUserDefaultsExample (architecture arm64):
Contents of (__DATA,__objc_classlist) section
00000001000081f8 0x1000090e0
           isa 0x1000090b8
    superclass 0x0
         cache 0x0
        vtable 0x0
          data 0x1000082f0 (struct class_ro_t *)
                    flags 0x184 RO_HAS_CXX_STRUCTORS
            instanceStart 8
             instanceSize 24
                 reserved 0x0
               ivarLayout 0x10000785f
                layout map: 0x02
                     name 0x100007852 SetUserPrefs
              baseMethods 0x100008270 (struct method_list_t *)
		   entsize 24
		     count 2
		      name 0x100006d79 setPrefs
		     types 0x10000789d v16@0:8
		       imp 0x100005a54
		      name 0x100006d82 .cxx_destruct
		     types 0x10000789d v16@0:8
		       imp 0x100005b7c
            baseProtocols 0x0
                    ivars 0x1000082a8
                    entsize 32
                      count 2
			   offset 0x1000090a0 68719476744
			     name 0x100006d90 username
			     type 0x1000078a5 @"NSString"
			alignment 3
			     size 8
			   offset 0x1000090a4 68719476752
			     name 0x100006d99 email
			     type 0x1000078a5 @"NSString"
			alignment 3
			     size 8
           weakIvarLayout 0x0
           baseProperties 0x0
Meta Class

......
```

Unfortunately this doesn't really give us a readable representation of the class, methods, and instance variable information that is mapped into the binary.  However, **class-dump-z** can solve all of our problems: 

```
[~/Tools/mobile/ios/class-dump-z/mac_x86]> ./class-dump-z ~/Development/python/mach-O-parser/NSUserDefaultsExample > dump
[~/Tools/mobile/ios/class-dump-z/mac_x86]> cat dump
/**
 * This header is generated by class-dump-z 0.2-0.
 * class-dump-z is Copyright (C) 2009 by KennyTM~, licensed under GPLv3.
 *
 * Source: (null)
 */

typedef struct _NSZone NSZone;

typedef struct CGPoint {
	float _field1;
	float _field2;
} CGPoint;

typedef struct CGSize {
	float _field1;
	float _field2;
} CGSize;

typedef struct CGRect {
	CGPoint _field1;
	CGSize _field2;
} CGRect;

@protocol NSObject
@optional
@property(readonly, copy) NSString* debugDescription;
@required
@property(readonly, copy) NSString* description;
@property(readonly, assign) Class superclass;
@property(readonly, assign) unsigned hash;
-(NSZone*)zone;
-(unsigned)retainCount;
-(id)autorelease;
-(oneway void)release;
-(id)retain;
-(BOOL)respondsToSelector:(SEL)selector;
-(BOOL)conformsToProtocol:(id)protocol;
-(BOOL)isMemberOfClass:(Class)aClass;
-(BOOL)isKindOfClass:(Class)aClass;
-(BOOL)isProxy;
-(id)performSelector:(SEL)selector withObject:(id)object withObject:(id)object3;
-(id)performSelector:(SEL)selector withObject:(id)object;
-(id)performSelector:(SEL)selector;
-(id)self;
-(Class)class;
-(BOOL)isEqual:(id)equal;
@end

@protocol UIApplicationDelegate <NSObject>
@optional
@property(retain, nonatomic) UIWindow* window;
-(void)application:(id)application didUpdateUserActivity:(id)activity;
-(void)application:(id)application didFailToContinueUserActivityWithType:(id)type error:(id)error;
-(BOOL)application:(id)application continueUserActivity:(id)activity restorationHandler:(id)handler;
-(BOOL)application:(id)application willContinueUserActivityWithType:(id)type;
-(void)application:(id)application didDecodeRestorableStateWithCoder:(id)coder;
-(void)application:(id)application willEncodeRestorableStateWithCoder:(id)coder;
-(BOOL)application:(id)application shouldRestoreApplicationState:(id)state;
-(BOOL)application:(id)application shouldSaveApplicationState:(id)state;
-(id)application:(id)application viewControllerWithRestorationIdentifierPath:(id)restorationIdentifierPath coder:(id)coder;
-(BOOL)application:(id)application shouldAllowExtensionPointIdentifier:(id)identifier;
-(unsigned)application:(id)application supportedInterfaceOrientationsForWindow:(id)window;
-(void)applicationProtectedDataDidBecomeAvailable:(id)applicationProtectedData;
-(void)applicationProtectedDataWillBecomeUnavailable:(id)applicationProtectedData;
-(void)applicationWillEnterForeground:(id)application;
-(void)applicationDidEnterBackground:(id)application;
-(void)application:(id)application handleWatchKitExtensionRequest:(id)request reply:(id)reply;
-(void)application:(id)application handleEventsForBackgroundURLSession:(id)backgroundURLSession completionHandler:(id)handler;
-(void)application:(id)application performFetchWithCompletionHandler:(id)completionHandler;
-(void)application:(id)application didReceiveRemoteNotification:(id)notification fetchCompletionHandler:(id)handler;
-(void)application:(id)application handleActionWithIdentifier:(id)identifier forRemoteNotification:(id)remoteNotification completionHandler:(id)handler;
-(void)application:(id)application handleActionWithIdentifier:(id)identifier forLocalNotification:(id)localNotification completionHandler:(id)handler;
-(void)application:(id)application didReceiveLocalNotification:(id)notification;
-(void)application:(id)application didReceiveRemoteNotification:(id)notification;
-(void)application:(id)application didFailToRegisterForRemoteNotificationsWithError:(id)error;
-(void)application:(id)application didRegisterForRemoteNotificationsWithDeviceToken:(id)deviceToken;
-(void)application:(id)application didRegisterUserNotificationSettings:(id)settings;
-(void)application:(id)application didChangeStatusBarFrame:(CGRect)frame;
-(void)application:(id)application willChangeStatusBarFrame:(CGRect)frame;
-(void)application:(id)application didChangeStatusBarOrientation:(int)orientation;
-(void)application:(id)application willChangeStatusBarOrientation:(int)orientation duration:(double)duration;
-(void)applicationSignificantTimeChange:(id)change;
-(void)applicationWillTerminate:(id)application;
-(void)applicationDidReceiveMemoryWarning:(id)application;
-(BOOL)application:(id)application openURL:(id)url sourceApplication:(id)application3 annotation:(id)annotation;
-(BOOL)application:(id)application handleOpenURL:(id)url;
-(void)applicationWillResignActive:(id)application;
-(void)applicationDidBecomeActive:(id)application;
-(BOOL)application:(id)application didFinishLaunchingWithOptions:(id)options;
-(BOOL)application:(id)application willFinishLaunchingWithOptions:(id)options;
-(void)applicationDidFinishLaunching:(id)application;
@end

@interface SetUserPrefs : XXUnknownSuperclass {
	NSString* username;
	NSString* email;
}
-(void).cxx_destruct;
-(void)setPrefs;
@end

@interface AppDelegate : XXUnknownSuperclass <UIApplicationDelegate> {
	NSManagedObjectContext* _managedObjectContext;
	NSManagedObjectModel* _managedObjectModel;
	NSPersistentStoreCoordinator* _persistentStoreCoordinator;
	UIWindow* _window;
}
@property(readonly, copy) NSString* debugDescription;
@property(readonly, copy) NSString* description;
@property(readonly, assign) Class superclass;
@property(readonly, assign) unsigned hash;
@property(readonly, assign, nonatomic) NSPersistentStoreCoordinator* persistentStoreCoordinator;
@property(readonly, assign, nonatomic) NSManagedObjectModel* managedObjectModel;
@property(readonly, assign, nonatomic) NSManagedObjectContext* managedObjectContext;
@property(retain, nonatomic) UIWindow* window;
-(void).cxx_destruct;
-(void)saveContext;
-(id)applicationDocumentsDirectory;
-(void)applicationWillTerminate:(id)application;
-(void)applicationDidBecomeActive:(id)application;
-(void)applicationWillEnterForeground:(id)application;
-(void)applicationDidEnterBackground:(id)application;
-(void)applicationWillResignActive:(id)application;
-(BOOL)application:(id)application didFinishLaunchingWithOptions:(id)options;
@end

__attribute__((visibility("hidden")))
@interface ViewController : XXUnknownSuperclass {
}
-(void)didReceiveMemoryWarning;
-(void)viewDidLoad;
@end
```

Now we have a much cleaner representation of the class, method, and instance variable information: 

```
@interface SetUserPrefs : XXUnknownSuperclass {
    NSString* username;
    NSString* email;
}
-(void).cxx_destruct;
-(void)setPrefs;
@end
```
