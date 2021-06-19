+++
tags = ["writeup"]
title = "the hamming (7, 4) error-correcting code"
index = "0006"
slug = "0006"
layout = "default"
+++
One of my assignments in my freshman year at UTK was a program implementing the hamming (7, 4) error-correcting code. I'm not really sure why, but I implemented this code three more times in three different languages. Here's a couple thoughts/notes I have on implementing the code:

### **1. Background**
Firstly, the math behind the code is described rigorously on the [wikipedia article](https://en.wikipedia.org/wiki/Hamming(7,4)). The idea is that, for every 4 bits of data, you encode an additional 3 **parity bits** alongside the data to allow single-bit errors to be detected **and** corrected. The youtube channel, [3blue1brown](https://www.youtube.com/c/3blue1brown) has a pretty wonderful [video explanation](https://www.youtube.com/watch?v=X8jsijhllIA) of how all this works. We're not too concerned with *how* it works though and moreso how to implement it.

### **2. Some shortcuts**
For an actual implementation of the code, there are a number of shortcuts you can take: 

1. There are only 16 possible combinations of parity bits, since there are only 16 possible combinations of 4 bits. No need to calculate the parity for some data if we just precompute it and store it in an array we can index:
{{< highlight cpp >}}
const char PARITY[] = {
  0x0, 0x3, 0x5, 0x6,
  0x6, 0x5, 0x3, 0x0,
  0x7, 0x4, 0x2, 0x1,
  0x1, 0x2, 0x4, 0x7
};
{{< /highlight >}}
So the parity for any 4 data bits can be obtained by `const char p = PARITY[databits]`

1. The same way there are only 16 possible sets of parity bits, there are only 16 possible codewords total. This isn't necessary, but for my implementations, I used this table as well as the parity table.
{{< highlight cpp >}}
const char TRANSMIT[] = {
  0x00, 0x07, 0x19, 0x1e,
  0x2a, 0x2d, 0x33, 0x34,
  0x4b, 0x4c, 0x52, 0x55,
  0x61, 0x66, 0x78, 0x7f
};
{{< /highlight >}}
This way, any codeword can be obtained by accessing the array with 4 data-bits: {{< highlight cpp >}} codeword = TRANSMIT[databits] {{< /highlight >}}

3. There's no need to do any matrix multiplication like in the wikipedia article to extract the data and the parity. (even the shortcut for bit-matrix multiplication described there is more than what we need to do) We can just manipulate those out with some `AND`ing and some bit-shifting: 
{{< highlight cpp >}}
char extract_parity(const char b) {
  auto p2 = (b & 0x08) >> 1;
  auto p1 = b & 0x02;
  auto p0 = b & 0x01;

  return p2 | p1 | p0;
}

char extract_data(const char b) {
  auto d3 = (b & 0x40) >> 3;
  auto d2 = (b & 0x20) >> 3;
  auto d1 = (b & 0x10) >> 3;
  auto d0 = (b & 0x04) >> 2;

  return d3 | d2 | d1 | d0;
}
{{< /highlight >}}

4. And finally, detecting and correcting any flipped bit is a lot easier than you might think. Given an encoded byte, let `p1` be the parity bits retrieved from that byte via `extract_parity`. Let `p2` be the parity bits obtained by accessing the parity array at the value of the data bits obtained via `extact_data`. In other words:
{{< highlight cpp>}}
char p1 = extract_parity(byte);
char p2 = PARITY[ extract_data(byte) ];
{{< /highlight >}}
The `xor` of these two values actually **tells us the bit position that needs to be flipped + 1**. So our `correct` function can be written as:
{{< highlight cpp>}}
char correct(const char b, const char p1, const char p2) {
  auto diff = p1 ^ p2;
  return b ^ ((0x01 << (diff)) >> 1);
}
{{< /highlight >}}
This function takes a single 1-bit, shifts it to the **left by `diff`**, and then to the **right by 1**. It then returns the `xor` of our byte and that value. The reason we shift to the right is because `diff` is one more than the bit position we need to correct. **Note that if diff is 0, then b will be left unchanged, meaning you can execute this function on non-error bits**.

### **3. Conclusion**

That's about it for these notes; I may or may not update them later if I make any more progress. If you ever find yourself wanting to implement the hamming (7, 4) error correcting code, take a look at my (possibly good?) implementations: 

* C++ implementation: https://github.com/s-viour/hamming74cpp
* Rust implementation: https://github.com/s-viour/hamming74rs
* Haskell implementation: https://github.com/s-viour/hamming74hs