# Shhhloader
Shhhloader is a work in progress shellcode loader. It takes raw shellcode as input and compiles a C++ stub that does a bunch of different things to try and bypass AV/EDR. The included python builder will work on any Linux system that has Mingw-w64 installed.

**1/25/24 EDIT: Shhhloader now supports [PoolParty](https://github.com/SafeBreach-Labs/PoolParty) variant 7! An option was also added that will combine PoolParty with Module Stomping to prevent the executed shellcode from residing in unbacked memory. See below for a video demonstrating this new injection method against Defender ATP:**

<details>
  <summary>See Video</summary>
<video src="https://github.com/icyguider/Shhhloader/assets/79864975/db5da0c5-8589-4abc-9292-fe028031c88d"></video>
</details>

```
┳┻|
┻┳|
┳┻|
┻┳|
┳┻| _
┻┳| •.•)  - Shhhhh, AV might hear us! 
┳┻|⊂ﾉ   
┻┳|
usage: Shhhloader.py [-h] [-p explorer.exe] [-m QueueUserAPC] [-u] [-w] [-nr] [-ns] [-l] [-v] [-sc GetSyscallStub] [-d] [-dp apphelp.dll] [-s domain]
                     [-sa testlab.local] [-o a.exe] [-pp explorer.exe] [-ppv] [-np] [-cp] [-td ntdll.dll] [-ef NtClose]
                     file

ICYGUIDER'S CUSTOM SYSCALL SHELLCODE LOADER

positional arguments:
  file                  File containing raw shellcode

options:
  -h, --help            show this help message and exit
  -p explorer.exe, --process explorer.exe
                        Process to inject into (Default: explorer.exe)
  -m QueueUserAPC, --method QueueUserAPC
                        Method for shellcode execution (Options: PoolPartyModuleStomping, PoolParty, ThreadlessInject, ModuleStomping, QueueUserAPC,
                        ProcessHollow, EnumDisplayMonitors, RemoteThreadContext, RemoteThreadSuspended, CurrentThread) (Default: QueueUserAPC)
  -u, --unhook          Unhook NTDLL in current process
  -w, --word-encode     Save shellcode in stub as array of English words
  -nr, --no-randomize   Disable syscall name randomization
  -ns, --no-sandbox     Disable sandbox checks
  -l, --llvm-obfuscator
                        Use Obfuscator-LLVM to compile stub
  -v, --verbose         Enable debugging messages upon execution
  -sc GetSyscallStub, --syscall GetSyscallStub
                        Syscall execution method (Options: SysWhispers2, SysWhispers3, GetSyscallStub, None) (Default: GetSyscallStub)
  -d, --dll             Generate a DLL instead of EXE
  -dp apphelp.dll, --dll-proxy apphelp.dll
                        Create Proxy DLL using supplied legitimate DLL (File must exist in current dir)
  -s domain, --sandbox domain
                        Sandbox evasion technique (Options: sleep, domain, hostname, username, dll) (Default: sleep)
  -sa testlab.local, --sandbox-arg testlab.local
                        Argument for sandbox evasion technique (Ex: WIN10CO-DESKTOP, testlab.local)
  -o a.exe, --outfile a.exe
                        Name of compiled file

PPID Spoofing:
  -pp explorer.exe, --ppid explorer.exe
                        Parent process to use for PPID Spoofing (Default: explorer.exe)
  -ppv, --ppid-priv     Enable spoofing for privileged parent process (Disabled by default)
  -np, --no-ppid-spoof  Disable PPID spoofing

ThreadlessInject:
  -cp, --create-process
                        Create process instead of injecting into existing one
  -td ntdll.dll, --target-dll ntdll.dll
                        Target DLL containing export function to overwrite
  -ef NtClose, --export-function NtClose
                        Export function to overwrite

```

**Features:**
* 10 Different Shellcode Execution Methods (PoolParty, PoolPartyModuleStomping, ThreadlessInject, ModuleStomping, QueueUserAPC, ProcessHollow, EnumDisplayMonitors, RemoteThreadContext, RemoteThreadSuspended, CurrentThread)
* PPID Spoofing
* Block 3rd Party DLLs
* Unhook NTDLL via KnownDLLs
* SysWhispers2, SysWhispers3, & GetSyscallStub
* API Hashing for SW2 & SW3
* Compile-Time String Encryption
* Obfuscator-LLVM (OLLVM) Support 
* Automatic DLL Proxy Generation
* Havoc C2 Framework Integration
* Syscall Name Randomization
* Store Shellcode as English Word Array
* XOR Encoding with Dynamic Key Generation
* Sandbox Evasion via Loaded DLL, Domain, User, Hostname, and System Enumeration

See below for a PoC video of the ThreadlessInject method being used to inject a Havoc beacon into IE without generating any alerts and minimal events in Microsoft Defender for Endpoint (MDE) EDR (Recorded 4/3/2023):
<details>
  <summary>See Video</summary>
<video src="https://github.com/icyguider/Shhhloader/assets/79864975/27b8805f-88f6-442f-aa62-da0bf8919fc3"></video>
</details>

See below video demonstrating the Module Stomping injection technique executed via DLL Proxying (Recorded 8/9/2022):
<details>
  <summary>See Video</summary>
<video src="https://github.com/icyguider/Shhhloader/assets/79864975/a692e7ae-c9af-453e-8c6b-7132c9d75444"></video>
</details>

Shhhloader has also been integrated into the [Havoc C2 Framework](https://github.com/HavocFramework/Havoc) via a module by [@jakobfriedl](https://github.com/jakobfriedl). See below for a video demonstrating the module:

<details>
  <summary>See Video</summary>
<video src="https://github.com/icyguider/Shhhloader/assets/79864975/e1b7db2b-6c8e-44b4-8996-7d98a5fe8386"></video>
</details>

**Known Issues/Notes:**
* The PoolParty injection methods are currently configured to inject into and existing process. Therefore, you must use the `-p` flag to supply the name of a process that you know or suspect is already running on the target system.
* The PoolParty injection methods will not work with large shellcode files like sliver (~11MB). 
* The ThreadlessInject shellcode execution method must currently inject into a non-suspended process. This is done automatically for you if the "-cp" option is supplied. I hope to figure out a way to get it working with suspended processes soon.
* The Havoc C2 module currently hangs when specifying a Listener instead of a raw shellcode file. This will be fixed soon hopefully.
* Windows Defender will detect most files generated by this tool, so please do not post an issue saying "DETECTED!!!". Play around with the new options and features until you get something that works; they were added for a reason :). Executing the generated file in memory is also a good way to evade these detections.
* Obfuscator-LLVM (OLLVM) is not compatible with SW2 & SW3 due to how clang handles inline assembly. There is currently no plan to make them compatible; checks have been added to prevent the user from trying to use these features together.
* There are **a ton** of bugs in my code. Please test everything in advance before using for something important, and PLEASE provide as much information as possible when opening an issue. (THANKS!)

**Planned Updates:**
* Hardware Breakpoint (HWBP) syscall option
* Create processess using syscall instead of standard WinAPI function
* Integrate ThreadlessInject with other shellcode injection techniques (Ex. ModuleStomping)
* Refactor code to be more modular and readable
* Fix bugs in Havoc C2 module
* Fix countless bugs by learning to program better ;)

**OPTIONAL:** To use the [Obfuscator-LLVM](https://github.com/heroims/obfuscator) flag, you must have it installed on your system alongside [wclang](https://github.com/tpoechtrager/wclang). I've found this to be a bit of a pain but you should be able to do it with a little perseverance. Here's a step-by-step that I used to install the llvm-13.x branch of OLLVM on my Kali Linux system:
<details>
  <summary>See Details</summary>
  
```
# Clone and Run CMake
git clone -b llvm-13.x https://github.com/heroims/obfuscator.git
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_NEW_PASS_MANAGER=OFF ../obfuscator/llvm/

# Configure CMake and Compile OLLVM
export clang_build_dir=$(cd ../; pwd)/obfuscator/clang
sed -i 's/LLVM_TOOL_CLANG_BUILD:BOOL=OFF/LLVM_TOOL_CLANG_BUILD:BOOL=ON/g' CMakeCache.txt
sed -i "s|LLVM_EXTERNAL_CLANG_SOURCE_DIR:PATH=|LLVM_EXTERNAL_CLANG_SOURCE_DIR:PATH=$clang_build_dir|g" CMakeCache.txt
make -j7

# ONCE COMPILED, BACKUP ORIGINAL CLANG BINARIES
mv /usr/bin/clang /usr/bin/clang13.0.1
mv /usr/bin/clang++ /usr/bin/clang++13.0.1

# Then in OLLVM build/bin dir, copy the newly build clang bins
cp bin/clang /usr/bin/clang
cp bin/clang++ /usr/bin/clang++

# Then install wclang
cd ..
git clone https://github.com/tpoechtrager/wclang.git
cd wclang/
cmake -DCMAKE_INSTALL_PREFIX=_prefix_ .
make
make install
export wclang_path=$(pwd)/_prefix_/bin
echo "export PATH=$wclang_path:$PATH" >> ~/.bashrc
export PATH=$wclang_path:$PATH

# Then backup original lib files
cp -R /lib/llvm-13/lib/clang/13.0.1/include/ /lib/llvm-13/lib/clang/13.0.1/include_backup/

# Finally in the OLLVM build/bin/lib/clang/13.0.1/ dir, copy the include folder
cd ../build/lib/clang/13.0.1/
cp -R include/ /lib/llvm-13/lib/clang/13.0.1/
```
</details>

There is probably a better way to do this but this is what worked for me. If you have issues, just keep trying and ensure that you can run `x86_64-w64-mingw32-clang++ -v` and it contains either "Obfuscator-LLVM" or "heroims" in the output. Unfortunately I do not have the time to assist individuals who may need more help, but you can try reading [this issue](https://github.com/icyguider/Nimcrypt2/issues/6) on my Nimcrypt2 repo where a couple of users figured out how to do it on their systems.

**Greetz & Credit:**
* [@Jackson_T](https://twitter.com/Jackson_T) for his amazing project SysWhispers: https://github.com/jthuraisamy/SysWhispers
* [@FalconForceTeam](https://twitter.com/falconforceteam) for their syscall generation tool that supports SysWhispers2: https://github.com/FalconForceTeam/SysWhispers2BOF
* [@snovvcrash](https://twitter.com/snovvcrash) for their DInjector project, which I used as a template for many of the included injection techniques: https://github.com/snovvcrash/DInjector
* [@Cerbersec](https://twitter.com/cerbersec) for their Ares project, whose code I used for PPID Spoofing, Blocking 3rd Party DLLs and Sandbox Evasion: https://github.com/Cerbersec/Ares
* [@spotheplanet](https://twitter.com/spotheplanet) for their blog post on Retrieving ntdll Syscall Stubs from Disk at Run-time: https://www.ired.team/offensive-security/defense-evasion/retrieving-ntdll-syscall-stubs-at-run-time
* [@_RastaMouse](https://twitter.com/_RastaMouse) for his code and article on Module Stomping which I ported to C++: https://offensivedefence.co.uk/posts/module-stomping/
* [@s4ntiago_p](https://twitter.com/s4ntiago_p) for their NanoDump project and the [randomize_sw2_seed.py](https://github.com/helpsystems/nanodump/blob/main/scripts/randomize_sw2_seed.py) script in particular: https://github.com/helpsystems/nanodump
* [@skadro-official](https://github.com/skadro-official) for their skCrypter project that this tool utilizes for compile-time string encryption: https://github.com/skadro-official/skCrypter
* [@\_EthicalChaos\_](https://twitter.com/_EthicalChaos_) for their ThreadlessInject project: https://github.com/CCob/ThreadlessInject
* [0xLegacyy](https://twitter.com/0xLegacyy) for their BOF version of ThreadlessInject: https://github.com/iilegacyyii/ThreadlessInject-BOF
* [@D1rkMtr](https://twitter.com/D1rkMtr) for their ntdll unhooking collection project: https://github.com/TheD1rkMtr/ntdlll-unhooking-collection
* [@KlezVirus](https://twitter.com/KlezVirus) for their SysWhispers3 project: https://github.com/klezVirus/SysWhispers3
* [@jakobfriedl](https://github.com/jakobfriedl) for making a Havoc C2 Module for Shhhloader
* [@_0xDeku](https://twitter.com/_0xDeku) for their research and code regarding process injection using Windows Thread Pools (PoolParty): https://github.com/SafeBreach-Labs/PoolParty
* [@0xEr3bus](https://twitter.com/0xEr3bus) for their PoolParty BOF which was used as a reference: https://github.com/0xEr3bus/PoolPartyBof
