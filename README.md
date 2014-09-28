```
╦  ┌─┐┬─┐┌─┐┌┐┌┌─┐┌┬┐┌┬┐┌─┐   ┌┬┐┬ ┬┌─┐    ┬  ┬┌┐ 
║  │ │├┬┘│ ┬│││├┤  │  │ ├┤     │ ├─┤├┤     │  │├┴┐
╩═╝└─┘┴└─└─┘┘└┘└─┘ ┴  ┴ └─┘    ┴ ┴ ┴└─┘    ┴─┘┴└─┘

```

`Lorgnette` is a library for locating symbols of any running process on OS X. 
It acts just like `dlsym()` but not limited to the current process, allowing you to examine
any running program on your system.   


> If you're wondering whether there's any other way to accomplish this task: **yes, there's one!**
OS X contains a private Apple framework called `CoreSymbolication` that can be used to locate symbols
in any running task on the system and even more.  
So if you need something production-ready (heh), you should use the Apple thing instead of `liblorgnette`.  
See [`CoreSymbolication`](#coresymbolication) section of this file.  

### Usage

If the target is a `mach_task_self()` then `lorgnette_lookup()` will act like `dlsym()`. 
But unlike `dlsym()` it can be used to locate not-exported symbols.  

```c
#include "lorgnette.h"
mach_vm_address_t main_addr = lorgnette_lookup(mach_task_self(), "main");

assert(dlsym(RTLD_DEFAULT, "main") == main_addr);
```

Otherwise it will inspect any foreign task you have rights to control (`task_for_pid` isn't
for everyone, you know):  

```c
pid_t proc = 20131;

task_t target;
task_for_pid(mach_task_self(), proc, &target);

mach_vm_address_t remote_addr = lorgnette_lookup(target, "_private_function");
```


### Interface  

**`mach_vm_address_t lorgnette_lookup(task_t target, const char *symbol_name);`**    

Lookup a symbol adress inside an arbitrary process' address space.

> This function iterates *local symbols first* and only then it looks for symbols in
linked libraries.  

| Parameter   | Type (in/out) | Description |
| :--------: | :-----------: | :---------- |
| `target` | in  | _**(required)**_ The target process to inspect  |  
| `symbol_name` | in| _**(required)**_ The name of the symbol to find. This parameter must not be NULL  |  


| Return value  |  
| :---------- |   
| An address of the given symbol within the given process, or 0 (zero) if this symbol could not be found |  


## CoreSymbolication  

Here're some reverse engineered headers for the framework along with functionality tests: [mountainstorm/CoreSymbolication](https://github.com/mountainstorm/CoreSymbolication).  
Also see [this StackOverflow question](http://stackoverflow.com/questions/17445960/finding-offsets-of-local-symbols-in-shared-libraries-programmatically-on-os-x) by Johannes Weiß.

---------

If you found any bug(s) or something, please open an issue or a pull request — I'd appreciate your help! (^,,^)

Dmitry Rodionov, 2014  
i.am.rodionovd@gmail.com