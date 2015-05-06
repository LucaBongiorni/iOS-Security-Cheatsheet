Binary Enumeration
------------------

First we need to figure out if we are dealing with a Universal 'FAT' binary, or Macho-O: 

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

If the application came from the Apple Store, then we know it is probably encrypted.  We can use otool to read the ```cryptid``` field within the ```LC_SEGMENT```:

```
[~/Development/python/mach-O-parser]> otool -lV NSUserDefaultsExample | grep 'cryptid'
      cryptid 0
      cryptid 0
```

Our binaries are not encrypted, because they are ad-hoc distributions from XCode and not prepared for store deployment.
