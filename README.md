# Tyche-Tester

Tyche is the Greek goddess of fortune and luck.

### Introduction
----------------

This project contains code which concerns itself with testing the random walk hypothesis using the NIST cryptographic suite of tests for randomness. This project includes an interface to Quandl which makes it especially easy to test market *returns* data for randomness including, but not limited too, stock prices, interest rates, foreign exchange rates, and commodity prices.

### Dependencies
----------------

1. pandas - for general data processing
2. numpy - for general data processing
3. scipy - for the special functions
4. bitstring - for the IEEE 754 converter

### Contributors
----------------

1. [Stuart Reid](http://www.TuringFinance.com)

### Project Stucture
--------------------

The project contains the following Python source files,

1. **Main.py** - this file contains example code which shows how to download market data, convert that market data into one or more binary representations which can be used in conjunction with the NIST test suite, and how to test that binary data for randomness.
2. **DataDownloader.py** - this file contain code for downloading market data from Quandl.com and joining it together to produce a complete pandas DataFrame of market data devoid of missing values which can then be converted into a BinaryFrame and tested for randomness.
3. **BinaryFrame.py** - this file contains methods for converting a pandas DataFrame into a so-called BinaryFrame. The BinaryFrame stores the column data from the DataFrame as binary strings in a dictionary. Two unbiased converters are supported. These are discussed in more detail in the binary converstion methodologies section of this readme file.
4. **RandomnessTests.py** - this file contains intuitive and easy to understand implementations of each of the randomness tests presented in the NIST test suite. These tests are discussed in a lot of detail in the NIST Tests for Randomness section of this readme file.

In addition to these, the project allows a user to create a local .private.csv file which contains private information such as one's Quandl authentication token and login details if you are using the project from behind a proxy. The file can be structured as follows:

### Binary Conversion Methodologies
-----------------------------------

Two binary conversion methodologies are supported by the project. 

##### Discretization 

This method simply loops through the floating point data in the market data pandas DataFrame and converts it to binary by testing if the return is greater than, or less than, 0. If the return is greater than 0, the return is represented as a 1-bit whereas if the return is less than 0 it is represented as a 0-bit. In the special case that the return is exactly equal to 0, the return is represented as 01 which removes the otherwise small bias towards producing zero-bits.

##### Adapted IEEE 754 Converter

This method loops over the floating point returns data in the market data pandas DataFrame and converts it using the standard IEEE 754 method of converting floating point numbers to binary. This is done using the bitstring package. The problem with this is that the conversion is biased in two ways, firstly, because numbers are only differentiated by one sign bit but are are otherwise equivalent and, secondly, because zero is represented as a sequence of zeros which results in a bias towards zero bits. For example,

+-0.0 = 00000000000000000000000000000000<br />
-0.25 = 10111110100000000000000000000000<br />
+0.25 = 00111110100000000000000000000000

To overcome this if the floating point number was negative, the bits (excluding) the sign bit are flipped and any and all floating points equal to zero are replaced with an unbiased bitstring of alternating zeros and ones,

+-0.0 = 01010101010101010101010101010101<br />
-0.25 = 10111110100000000000000000000000<br />
+0.25 = 01000001011111111111111111111111

Assuming the floating point numbers are uniformly distributed between -1.0 and 1.0, this methodology results in an expectation that the number of ones and zeros in the resulting bitstring should be approximately equal. This is the condition under which the NIST tests for randomness should be applied. *Note: there may be other, yet undiscovered, challenges with this methodology*

### NIST Tests for Randomness
-----------------------------