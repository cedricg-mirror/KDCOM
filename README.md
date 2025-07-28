All credits to [BugChecker](https://github.com/vitoplantamura/BugChecker/tree/master/KDCOM)  

I was looking for a mean to boot Windows in debug mode (to deactivate patchguard) without having an actual kernel debugger attached.  

Booting in debug mode is not enough as PatchGuard is looking for the presence of an actual debugger to deactivate itself, as described in this paper [PatchGuard Analysis](https://blog.tetrane.com/downloads/Tetrane_PatchGuard_Analysis_RS4_v1.00.pdf)  

```asm
0xfffff803c98dabd1 movzx edx, byte ptr [rip – 0x4f3255] ; KdDebuggerNotPresent
0xfffff803c98dabd8 movzx eax, byte ptr [rip – 0x51ee66] ; KdPitchDebugger
0xfffff803c98dabdf or edx, eax
0xfffff803c98dabe1 mov ecx, edx
0xfffff803c98dabe3 neg ecx
0xfffff803c98dabe5 sbb r8d, r8d
0xfffff803c98dabe8 and r8d, 0xffffffee
0xfffff803c98dabec add r8d, 0x11
0xfffff803c98dabf0 ror edx, 1
0xfffff803c98dabf2 mov eax, edx
0xfffff803c98dabf4 cdq
0xfffff803c98dabf5 divide error while executing idiv r8d
```

I just changed the original BugChecker code to :  

-  remove the Hooking capabilities offered by "KdSetBugCheckerCallbacks" that were unnecessary to my use case  
-  remove the constant (fake) 'reminder' that a Kernel Debugger is attached as in this piece of code in "KdReceivePacket":  
```C
if (*KdDebuggerNotPresent)
	{
		*KdDebuggerNotPresent = FALSE;
		SharedUserData->KdDebuggerEnabled |= 0x2;
	}
```
-  changed the ReturnStatus in "KdReceivePacket" from DBG_CONTINUE to DBG_EXCEPTION_HANDLED  

Installation procedure can be found on the page of the original author but really comes down to :   

- Disabling Secure Boot
- Replacing the original kdcom.dll in system32 by this version
- Adding a boot entry in debug mode : bcdedit.exe /debug {entry} on  
- Activating testsigning : bcdedit /set {entry} testsigning on  
- Specifying the debug mode : bcdedit /set {entry} dbgtransport kdcom.dll and bcdedit /set {entry} debugtype serial  
- For convenience changing the boot menu layout : bcdedit /set {default} bootmenupolicy legacy  

   

 
