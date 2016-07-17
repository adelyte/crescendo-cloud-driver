# Crescendo Cloud Driver
[Crescendo Cloud](https://crescendo.cloud) driver for Crestron control systems

## Getting Started
Due to quirks with SIMPL+, the driver is divided into 2-Series and 3-Series folders. The `Crescendo Cloud.usp` files are identical in both folders, but the `Crescendo Cloud.ush` header files are scoped to each processor type.

To keep the module inputs, outputs, and parameters neatly aligned, the driver is not wrapped in a user macro, which would force all the parameters to the bottom of the module. However, there a few SIMPL symbols the driver depends on that need to be added to the program along with the driver.

1. Clone or download this repository.
2. Choose the `2-Series/` or `3-Series/` processor.
3. Copy `Crescendo Cloud.usp` to your program directory, which will make it available as a project module.
4. Open `Crescendo Cloud.smw` in SIMPL Windows.
5. Copy the Crescendo Cloud folder from the Crescendo Cloud program.
6. Paste the folder into your program.

This will provide a completely linked module, neatly aligned and ready for custom integration.

### 2-Series

## Performance
The Crescendo Cloud driver is a high-performance standalone SIMPL+ module. With default settings, it supports 198 `digital`, `analog`, and `serial` key:value pairs. These can be reduced for a (very slight) performance increase or expanded if more values are needed:

```c
// Input/Output Sizes
#DEFINE_CONSTANT #KEY_VALUE_SIZE   198
#DEFINE_CONSTANT #KEY_VALUE_SIZEx2 396
```

Values are keyed for transmission using a simple array index and transmission performance is not affected by the number of key:value pairs.

```c
Digital_Value_Is[197] = 1; // index is 197, key is Digital_Key[197]
```

Values are looked up on receipt using a hash table of keys. This hash table is extremely performant (over 2,100 ops/sec on 3-Series and 1,600 ops/sec on 2-Series). More importantly, hash table performance does not degrade if the size of the hash table increases.

## Security
For performance reasons, the driver does not use encryption or checksums. Enterprise-grade encryption between the client network and Crescendo Cloud is available via IPSec, which should be configured on the client network equipment. Checksums are delegated to TCP, which includes a robust 16-bit checksum on every segment. 

### Password
A future version of the driver may include an optional password. Further discussion is available on [Issue #1](https://github.com/adelyte/crescendo-cloud-driver/issues/1).

#### ID as Password
The processor model, serial number, and MAC address are concatenated to form a processor ID, which functions as a password. 

Our processor sample set has a range of over 30 processor models (2<sup>5</sup>), 10 million serial numbers (2<sup>22</sup>), and 4 million MAC addresses (2<sup>21</sup>). Entropy is reduced by a correlation between the uppermost four bits of the MAC address with a serial number (-2<sup>4</sup>). A naive adversary wishing to exploit a random processor out of _n_ total processors would have to to search 2<sup>5</sup> &times; 2<sup>22</sup> &times; 2<sup>21</sup> &divide; 2<sup>4</sup> &divide; _n_ &divide; 2 = 2<sup>5 + 22 + 21 - 4 - log<sub>2</sub>_n_ - 1</sup> = 2<sup>43 - log<sub>2</sub>_n_</sup> locations on average for a single exploit.

> NOTE: 2-Series processors do not have serial numbers accessible from the Crestron operating system, see [Issue #2](https://github.com/adelyte/crescendo-cloud-driver/issues/2).

A well-informed adversary could exploit a specific processor easily, but it is not clear that Crescendo Cloud creates <em>additional</em> security risk against such an adversary, especially an adversary with physical access to the processor.
