
>[!Introduction]
>As we now from the [[Introduction to computer architecture|first lecture]], we know that memory organization also plays a vital part of the overall architecture.

![[memory-hierarchy-01.png]]

As we see, there are three layers of memory:
- The first one is the secondary memory, with the disk, these are very cheap, easy to produce and can be purchase at a large volume, but it takes forever to process the data (in term of processor's speed).
- The main memory (mostly known about RAM and ROM) - very quick, more expensive than the secondary memory, program is loaded into these memory, depends on the property of the program (temporary or must be persistent) RAM or ROM will involve.
- Lastly we have the cache - which acts as a bridge to narrow the gaps of speed between the processor (CPU) and the main memory. We also have many levels of cache, as shown in the above figure.

Virtual memory and cached are implemented to increase the efficiency of code executing - which helps improving the speed of the system. So how the memory was organized to achieve this? The answer is the memory-access locality. There are two types:
- Temporal locality - if a block of memory was accessed before, there is a high chance that block will be accessed again in the future.
- Spatial locality - if a block of memory was accessed before, there is a high chance that the related blocks (neighbor blocks) will be accessed again in the future.
If a block of code exhibits a very good locality of reference - it is highly concluded that code will be executed rapidly. And on the other hand, if the very same code piece has so-so or poor locality, well, the speed will be slower and slower.

