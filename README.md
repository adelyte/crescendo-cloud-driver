# Crescendo Cloud Driver
[Crescendo Cloud](https://crescendo.cloud) driver for Crestron control systems

## Performance
The Crescendo Cloud driver is a high-performance standalone SIMPL+ module. With default settings, it supports 998 `digital`, `analog`, and `serial` key:value pairs. These can be reduced for a (slight) performance increase or expanded if more values are needed:

```c
// Input/Output Sizes
#DEFINE_CONSTANT #KEY_VALUE_SIZE    998
#DEFINE_CONSTANT #KEY_VALUE_SIZEx2 1996
```

Values are keyed for transmission using a simple array index and transmission performance is not affected by the number of key:value pairs.

```c
Digital_Value_Is[997] = 1; // index is 997, key is Digital_Key[997]
```

Values are looked up on receipt using a hash table of keys. This hash table is extremely performant (over 2,100 ops/sec on 3-Series and 1,600 ops/sec on 2-Series). More importantly, hash table performance does not degrade if the size of the hash table increases.

## Security
For performance reasons, the driver does not use encryption or checksums. Enterprise-grade encryption between the client network and Crescendo Cloud is available via IPSec, which should be configured on the client network equipment. Checksums are delegated to TCP, which includes a robust 16-bit checksum on every segment. 

### Password
A future version of the driver may include an optional password. Presently, the processor model, serial number, and MAC address are concatenated to form a processor ID, which functions as a password. A naive adversary wishing to exploit a random processor would have to pick a processor model and search 2<sup>20</sup> serial numbers and 2<sup>24</sup> MAC addresses for an expected 2<sup>20</sup> &times; 2<sup>24</sup> &divide; 2 = 2<sup>43</sup> > 9 trillion searches.

An informed adversary could exploit a specific processor easily, but it is not clear that Crescendo Cloud creates <em>additional</em> security risk, especially against an adversary with physical access to the processor. Further discussion is available on [issue #1](https://github.com/adelyte/crescendo-cloud-driver/issues/1).
