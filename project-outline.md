Project Outline: Implementation of CBOR for RIOT
======

Team: Kevin Funk, Jana Cavojska

Introduction
-------
A CBOR (Concise Binary Object Representation) implementation for RIOT will make it possible for devices running RIOT to exchange serialized, compressed binary objects, which in turn could serve several purposes, such as remote procedure calls, message passing, or generally exchanging C data items to be processed on another RIOT device.

Alternatives to CBOR include:

* Not compressing at all: bad for bandwidth, converting data structures to strings (serializing) would still need to be implemented, target system would still need to decode the serialized stream
* JSON: requires more bandwidth than CBOR
* XML: requires more bandwidth than JSON

Main problems
-------

1. Get our initial code to run on RIOT
2. Define API for our CBOR library
3. Implement and test the CBOR encoder and decoder for simple types
4. Implement and test the CBOR encoder and decoder for complex types
5. Test our implementation against other working implementations

Subtasks:
-------

### (1) Writing a simple module + header, placing them in the appropriate directories, calling them from within a RIOT process

### (2) Define API for our CBOR library

### (3) Implement serialization and deserialization functions for simple types (PODs, such as int, float, bool, ...).

1. Test the (de)serialization code against each other. Check if serializing + deserializing an object yields the same object.

### (4) Implement serialization and deserialization functions for complex types (such as arrays, maps)

#### Problems

For serialization, we need to know the size of the CBOR data upfront in order to save the serialized data.
The question is how to design the serialization API in order to guarantee a low memory footprint.

For simple data types we can estimate the needed amount of bytes, reserve a buffer of sufficient size and then write the serialized form into the buffer.

For complex types, such as arrays and maps, this will not work.

We can think of three approaches in order to solve this:

* Reserving a roughly estimated amount of buffer upfront which can hold the serialized data
* Use a two-phase approach
    * First phase: Go through the to-be-serialized C structure and find out how many bytes it would take once serialized.
    * Second phase: Allocate the amount of memory determined from phase one, and then serialize the data to this memory location.
* Incrementally re-allocate the buffer holding the serialized data while doing the serialization

### (5) Test our implementation against other working implementations

This includes testing the encoding / decoding of streams of arbitrary length, containing an arbitrary number of data items.

1. Testing the encoder against other implementations:
Finding another encoder, checking to see if it produces the same output as our encoder for a sample input.

2. Testing the decoder against other implementations:
Encoding something using a working encoder, then decoding it using our decoder. Decoder output should be identical to encoder input.

Approach
------

### Serializing/Deserializing POD types

Suggested API for **cbor.h** (for simple types):

    // POD: int
    /// Return bytes written into @p dest
    size_t cbor_serialize(char* dest, int* input)
    {
        // Write dest
    }
    /// Return bytes written into @p dest
    size_t cbor_deserialize(const char* src, int* dest)
    {
        // Write dest
    }
    // POD: float (...)
    // POD: bool (...)

    // Serialize:
    size_t finalSize = 0;
    char dest[16]; // reserve big enough buffer (upper bound)

    int const i = 42;
    float f = 0;
    size_t offset = cbor_serialize(dest, &i);
    cbor_serialize(dest+offset, &f);

    // Deserialize:
    const char* src = "...";
    int i = 0;
    float f = 0;
    size_t offset = cbor_deserialize(src, &i);
    cbor_deserialize(src+offset, &f);

Timeline/Schedule
------

### Milestone 1: May 13th

* Have our initial code integrated in the RIOT codebase
* Define an API for the CBOR library that satisfies requirements for the RIOT-OS
    * Define preliminary API for (de)serializing simple types
    * Define preliminary API for (de)serializing complex types
* Implement the serialization and deserialization of at least one simple type
* Have a test infrastructure that allows to test the (de)serialization

### Milestone 2: June 17th

* Be able to serialize and deserialize all simple types
* Be able to parse byte streams containing an arbitrary number of encoded simple types

### Milestone 3: July 1st

* Be able to serialize and deserialize all complex types
* Be able to parse byte streams containing an arbitrary number of encoded simple and / or complex types

### Milestone 4: July 15th

* The last two weeks are meant as a buffer in case anything goes wrong and we get behind

Preliminary task assignement to team members:
------
* Get our initial code to run on RIOT; create a CBOR branch in the RIOT github repo: *Kevin*
* Define API for our CBOR library: *Kevin and Jana*, since this task requires coordination and discussion
* Implement and test the CBOR encoder and decoder for:
	* unsigned integers, negative integers: *Jana*
	* byte strings, unicode strings: *Kevin*
	* arrays of data items: *Jana*
	* maps of pairs of data items: *Kevin*
	* floating-point numbers: *Jana*
	* contentless simple data types, the "break" stop code: *Kevin*


References
------
* CBOR RFC: <https://tools.ietf.org/html/rfc7049>
* CBOR implementations in other languages: <http://cbor.io/impls.html>
* CBOR implementation in C++: <http://sourceforge.net/p/s11n/svn/HEAD/tree/c11n/include/s11n.net/c11n/c11n.h>
* RIOT OS source code: <https://github.com/RIOT-OS/RIOT/>
* RIOT OS wiki: <https://github.com/RIOT-OS/RIOT/wiki>

