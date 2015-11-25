### First class continuations in UE-4 Blueprints

UE-4 Blueprints have a concept called "Latent Actions", one of the simplest being _Delay_ shown below which suspends 
the current Blueprint for the specified number of seconds and then continues with whatever is connected to the _Completed_ pin. 
![](https://lh3.googleusercontent.com/-K-fzGqHab-A/VlUiyrcixHI/AAAAAAAAA6I/27OmbVKfDdI/w530-h392-p-k-rw/Delay.PNG) 

Currently all such latent actions show that clock icon. 
Here is another example from a plugin, which suspends the script during a network operation and then continues when the result is 
available: 
![](https://lh3.googleusercontent.com/-lmT6Fe0bGcU/VlUmI7HJLgI/AAAAAAAAA6Y/0eLKYlCg-Vw/w530-h200-p-k-rw/ApplyURL.PNG)

In UE-4.10, latent action nodes are only allowed at the top level of a Blueprint "event graph". They become inaccessible inside of 
functions. This is unfortunate because you are forced to use essentially global variables in such cases, and since an event graph 
contains a bunch of parallel event handlers, this quickly becomes a mess. 

Blueprint graphs are in fact compiled to byte-code which is interpreted by a virtual machine written in C++. This vm uses an 
explicit stack, vm functions being of the form FUNC(FFrame &Stack, void *Result). The 4.10 implementation creates the vm stack on 
the C++ call stack, using alloca to reserve space for the local variables. The explains why latent actions are only allowed at the 
top level, namely due to the fact that the program counter and nothing else is preserved during the callback process. By preserving
(a copy of) the entire vm call stack, this limitation can be removed. Note that local variables are shared between any such copies,
only the program counter of each frame is distinct. Now the copy must be allocated on the C++ heap rather than its stack in order to
persist until the callback occurs.

I've created a fork of 4.10 that makes the rather small changes necessary to enable the general use of latent actions.
In addition, it contains some experimental Blueprint nodes to enable using continuations directly in blueprints. 
Continuations in a sense generalize function "Return" nodes and in fact can be used in place of them. 
Here is an example:
