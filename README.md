Introduction:
This is my attempt of Project 1 Bit Hacks from MIT's 6.172 Performance Engineering of Software Systems. The specs of this project can be found in detail here: https://ocw.mit.edu/courses/6-172-performance-engineering-of-software-systems-fall-2018/resources/project-1-bit-hacks/ The gist of it was that we want to rotate a segment of a bitstring as fast as possible, without using parallel programming.


How to run:
Simply run "make test", and then "./everybit -l".


Methodology and Results:
On my laptop (2023 Macbook Pro), the original naive solution that used a for loop to iteratively shift the bitstring one by one was able to do bitstring rotations up to Tier 19 (3KB bitstring).

An improved solution used a different algorithm, specified in the description of the project. The general idea was to notice that a rotation was bound to split part of the bitstring into the form A|B, and the resulting rotation should produce the substring as B|A. Knowing this, and the fact that B|A = (A^R|B^R)^R, we can in turn transform this problem into reversing substrings. A naive solution that swapped bit by bit was inspired by the method, resulting in Tier 38 being completed (31MB bitstring).

The solution above didn't utilise word-level parallelism (if we can do 1 LOAD and 1 STORE to get and set 8 bytes, we shouldn't just waste the rest of the 63 bits), and doesn't take advantage of the hardware prefetcher. The solution in this case reversed the bitstring 8 bytes a time, resulting in drastically better performance, with Tier 46 being completed (1GB bitstring). A little help from AI was used to get the specific offsets and indices right. This is more than 300,000 times better than the first solution.


Additional considerations:
1. I tried to do 128 bits a time, using the __uint128_t, but the performance didn't really improve. I think that might be because the hardware registers can only hold 64 bits, so doing this doesn't really help. It would probably be better if I could somehow tell the hardware to use its vector registers, but the swapping operation isn't "vectorise-friendly".
2. The program that I have now doesn't really parallelise well (although the specs say no multicore stuff, I'd still like to explore this option). Further, due to the access pattern, there could still be quite a few cache misses for long bitstrings. In the future, I would like to try implementing a recursive bitstring reversal algorithm (this is probably cleaner than tiling), where we recursively reverse each half and concatenate them in opposite order. The threshold bitstring size can then be tuned for better performance. 
3. I've tested many of the other approaches online for this project, but I have yet to see a version that works better, at least on my Mac. However, according to some statistics I found, a certain group from MIT was able to achieve Tier 48 (4GB bitstring). This is really impressive considering it was on an Intel Xeon E5-2666 v3 machine running in the cloud, which isn't as fast as the 2023 Macbook Pro that I have. Perhaps they found a better algorithm?