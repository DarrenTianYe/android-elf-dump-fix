# elf-dump-fix
This resp include two utils about dumping and fixing arm elf32/elf64 from the memory.

- dump  
  - Run on android, can dump ELF from a process memory and fix it, Rebuild the Section Header for better IDA analysis.
- sofix
  - Run on PC, can fix an ELF file dumped from memory and rebuild the Section Header for better IDA analysis.

The main target is to rebuild the Section Header of an ELF by memory dumped.Useful in breaking packed so file like UPX or something like 360 libjiagu.so


## Build
 - dump
   - ```
     cd app/jni/
     ndk-build
      ```
   - output path is app/libs/armeabi-v7a/dump
   
 - sofix
   - on linux/mac, make sure clang/gcc is installed, just run ./build-fix.sh
   - on windows, It can be built in mingw , but not tested.
   
## HowToUse
 - sofix
   - params <src_so_path> <base_addr_in_memory_in_hex> <out_so_path>
     - <src_so_path> the elf file dumped from memory.(you can use dd or IDA debugger dumping an ELF file from android process)
     - <base_addr_in_memory_in_hex> the memory base for the elf file dumped from memory, if you don't know, pass 0 is ok
     - <out_so_path> the output file
   - example
     - ./sofix dumped.so 0x6f5a4000 b.so
 - dump
   - This is run on Android Phone
   - make sure your phone have root access.
   - push it onto /data/local/tmp and grant +x like this
     - adb push app/libs/armeabi-v7a/dump /data/local/tmp/ && adb shell chmod 777 /data/local/tmp/dump
   - use adb shell to enter your phone and switch to root user by su command.
   - params <pid> <base_hex> <end_hex> <outPath> [is-stop-process-before-dump] [is-fix-so-after-dump]
     - <pid> the process id you want to dump
     - <base_hex> the start address of ELF you want to dump in process memory, you can get this by ```cat /proc/<pid>/maps```
     - <end_hex> the end address of ELF you want to dump in process memory, you can get this by ```cat /proc/<pid>/maps```
     - <outPath> the fixed ELF output path in your phone.
     - [is-stop-process-before-dump] 0/1 should send sigal to the process before doing dump job, useful in some anti dumping app. if there is no anti dumping on your target process, 0 is ok
     - [is-fix-so-after-dump] 0/1 should do the fix job and Section Header rebuilding, if you pass on, it will try to fix the ELF after dump.
   - example
     - if you want to dump libc.so, and your /proc/[pid]/maps like this
     - ```
         40105000-4014c000 r-xp 00000000 b3:19 717        /system/lib/libc.so
         4014c000-4014d000 ---p 00000000 00:00 0 
         4014d000-4014f000 r--p 00047000 b3:19 717        /system/lib/libc.so
         4014f000-40152000 rw-p 00049000 b3:19 717        /system/lib/libc.so
         40152000-40160000 rw-p 00000000 00:00 0 
        ```
     - ./dump 1148 0x40105000 0x40160000 ./out.so 0 1
       - dump to 40160000 not 40152000 is because the ELF .bss memory if exist should be dump too, the fix process depends on it.
  
## Compare between no-fix and fixed ELF
![](imgs/no-fix.png)
![](imgs/fix.png)



'''
7848508000-7848612000 r-xp 00000000 103:2f 5961856                       /system/lib64/libcrypto.so
7848612000-7848627000 ---p 00000000 00:00 0
7848627000-7848638000 r--p 0010f000 103:2f 5961856                       /system/lib64/libcrypto.so
7848638000-7848639000 rw-p 00120000 103:2f 5961856                       /system/lib64/libcrypto.so
7848639000-784863a000 rw-p 00000000 00:00 0                              [anon:.bss]

./dump 10731 0x7848508000 0x784863a000 ./libcrypto.so 0 1
try dump 10731 from 0000007848508000 to 000000784863a000
try to read /proc/10731/mem fp:3, off=0000007848508000, sz=1253376
read return 1253376
1253376 writed
try fix ./libcrypto.so.tmp
fixed so has write to ./libcrypto.so
end fix ./libcrypto.so.tmp output to ./libcrypto.so


7765508000-7765548000 r-xp 00000000 103:33 143964                        /data/data/com.eg.android.AlipayGphone/app_SGLib/app_1672816243/main/libsgsecuritybodyso-6.5.15408830.so
7765548000-7765557000 ---p 00000000 00:00 0
7765557000-7765558000 r--p 0003f000 103:33 143964                        /data/data/com.eg.android.AlipayGphone/app_SGLib/app_1672816243/main/libsgsecuritybodyso-6.5.15408830.so
7765558000-77655ba000 rw-p 00040000 103:33 143964                        /data/data/com.eg.android.AlipayGphone/app_SGLib/app_1672816243/main/libsgsecuritybodyso-6.5.15408830.so
77655ba000-77655bf000 rw-p 00000000 00:00 0                              [anon:.bss]

7765680000-7765703000 rw-p 00000000 00:00 0                              [anon:jsi_v8]
7765780000-77657c0000 rw-p 00000000 00:00 0                              [anon:jsi_v8]
7765800000-7765801000 ---p 00000000 00:00 0                              [anon:libc_malloc]
7765801000-7765a00000 rw-p 00000000 00:00 0                              [anon:libc_malloc]
7765a00000-7765a01000 ---p 00000000 00:00 0                              [anon:libc_malloc]
7765a01000-7766800000 rw-p 00000000 00:00 0                              [anon:libc_malloc]
776681a000-776681b000 ---p 00000000 00:00 0                              [anon:thread stack guard]


7783ea7000-7783fa6000 r-xp 00000000 103:33 143893                        /data/data/com.eg.android.AlipayGphone/app_SGLib/app_1672816243/main/libsgmainso-6.5.15106478.so
7783fa6000-7783fb5000 ---p 00000000 00:00 0
7783fb5000-7783fb8000 r--p 000fe000 103:33 143893                        /data/data/com.eg.android.AlipayGphone/app_SGLib/app_1672816243/main/libsgmainso-6.5.15106478.so
7783fb8000-7783fed000 rw-p 00101000 103:33 143893                        /data/data/com.eg.android.AlipayGphone/app_SGLib/app_1672816243/main/libsgmainso-6.5.15106478.so
7783fed000-7783ffb000 rw-p 00000000 00:00 0                              [anon:.bss]


./dump 10731 0x7848508000 0x784863a000 ./libcrypto.so 0 1
./dump 29096 7765508000 77655bf000 ./libsgsecuritybodyso-6.5.15408830.new.so 0 1


'''

