# Program flow


The compiled library natlink.pyd/natlink.dll is registered with natspeak as a COM-Server support module.
 This causes our library to be loaded through the COM-Interface whenever Dragon started.
 During the initialization of the library a Python interpreter it started which in turn loads the natlink python modules.
 All of this only starts python process to actually talk to dragons natspeak we need to go back into the library. 
 Dragon itself provides a COM-Server which we can call from the C++ libary. 
 These calls are wrapped into a Python/C API by the same library that just started the python process. 
 So the python process pulls back into that very same library to talk to natspeak.
 
 **TODO** Currently I am not sure how callbacks when a grammar is recognised by natspeak are handled.
 
 ### Support module registration
 
 First we need to register our COM-Server with the windows registry.
 This is done by the installer. 
 Windows will associate the path of the libary with the GUID that we choose for Natlink (`dd990001-bb89-11d2-b031-0060088dc929`) which will set the ``HKEY_LOCAL_MACHINE\SOFTWARE\Classes\WOW6432Node\CLSID\{dd990001-bb89-11d2-b031-0060088dc929}\InprocServer32``for x64 systems.
 
 Then we need to inform dragon of the new module. To do so we add
  ```
[.Natlink]
App Support GUID={dd990001-bb89-11d2-b031-0060088dc929}
```
 to `C:\ProgramData\Nuance\NaturallySpeaking15\nsapps.ini` which also gives the module the identifier `.Natlink`
 
 We can then activate/deactivate our support Module by either adding or removing `.Natlink=Default` it from the 
 `[Global Clients]:` section of :  `C:\ProgramData\Nuance\NaturallySpeaking15\nssystem.ini`
 As far as I can tell the value to the `.Natlink` key is irrelevant.
 
 ### COM-Server implementation
``appsupp.h/appsupp.cpp``

Apparently we do not have access to the IDL (interface definition language) files. 

Indentation is important in the .reg file! I also remove my comments now .dll should work.

These files seem to implement the support interface defined in ``COM/dspeech.h``.

**TODO** The whole interface definition is littered with typdef. So I should go through and try to understand which methods are actually defined and what they do.
 
 ### DLL initialization and python Interpreter 
 ### COM-Python Wrapper
 
 The python natlink module is added to ``HKEY_CURRENT_USER\SOFTWARE\WOW6432Node\Python\PythonCore\3.8\PythonPath``
 as a Key/Subentry(?). This allows the sys module loader to find natlinkmain module.
 
 As this library is loaded as support-module the working directory during runtime is that of the dragon system and
 not the location of the library. Thus our python-module that we want to load from the library has to be
 found through the `sys.path`.