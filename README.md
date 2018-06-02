# Windows API resolution via hashing
Although this method of API obfuscation is relatively old, my friend who was wanting to increase his skills in the Windows sphere confronted me about a way a few malware families seem to resolve APIs. It's pretty simple, however he could not find any documentation with a solid programming example on the matter at the time, so I thought I'd quickly write something up regarding it. I *was* going to write my own loader for this example (loading the desired module via `LdrLoadDll` within `kernel32.dll`, walking the `InMemoryOrderModuleList` to find the desired loaded module, finding the exported function we're after within the EAT..) - however I thought this might of have been a bit overkill for such a simple concept, I want to cover writing your own PE loader in the future though as it's an interesting subject.

If you're not familiar with the PE format, take a look at this diagram from Microsoft - it outlines the way in which the table is structured. When `EAT` is mentioned, we're referring to the export address table.

![EAT diagram](https://i.imgur.com/u9RXQQd.gif)

So, every DLL in theory **should** have an export table where the functions that it implements are exposed to the loader - in order for an application for example to use one of the exported functions. This information resides in the EAT (export address table).

When you traditionally compile an executable and implicitly use for example `MessageBox`, the compiler will add this to this to the IAT, this will contain the module (DLL) that the symbol resides in too.

But, we're talking about obfuscation here right? What if someone doesn't want an analyst to see this in the export table? This is where obfuscated resolution comes in. At runtime, we can load these functions on the fly given a name - meaning they won't reside in this pesky IAT thus making it harder to analyse (kinda ;)).

This is now how API hashing works, API hashing is when we walk a given module's EAT looking for the **name** of a symbol which **matches our hash** we computed for it earlier. Once we get the address, we can freely use a function pointer and use the API just as normal.

In my example, I've decided not to use a cryptographically safe hashing routine as we just want a hashing algorithm which isn't too computationally expensive and fast, but at the same time produces unique enough results. I've chosen to use `SuperFastHash` created by Paul Hsieh. Again, this is a trivial example of how API hashing is used in the real world, it's not meant to be a complex example!
