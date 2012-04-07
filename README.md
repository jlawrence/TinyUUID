# TinyUUID #

## Summary ##

A JavaScript library that generates v4 UUIDs using the XXTEA block cipher, along with Math.random, as a pseudorandom data source

## Details ##

This JavaScript library generates UUIDs (Universally Unique IDentifiers), also known as GUIDs (Globally Unique IDentifiers). UUIDs are useful for identifying objects because they can be generated by independent programs with an extremely small probability of collision. [RFC 4122](http://www.ietf.org/rfc/rfc4122.txt) defines the standards for UUIDs. A UUID is 128 bit number, and it has a hexadecimal string representation of the form _xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_, where the _x_'s are the hexadecimal digits. There are several versions of UUIDs, and this library uses version 4. Version 4 UUIDs consist of pseudorandom data with certain bits set to indicate the version and variant.

This implementation does not use `Math.random` as a sole source for pseudorandom data. One reason is that web browsers use the timestamp to seed Math.random. Some seed it when the browser is opened, and others seed it when `Math.random` is called for the first time [1]. Therefore, if two browsers were opened (or `Math.random` was called for the first time) at the exact same time, and if the browsers performed the same number of calls to `Math.random`, then the generated numbers could end up being the same, resulting in UUID collisions. Furthermore, since the browser implementation of `Math.random` is often a linear congruential generator (LCG), the future and past output of `Math.random` can be deduced by observing some output [1]. This could possibly allow one to determine that several UUIDs were generate by the same client and to predict past and future UUIDs.

This library attempts to solve these problems by using a block cipher to generate random numbers. This would theoretically result in a cryptographically secure pseudo-random number generator (CSPRNG), but I would not suggest using this library for cryptography unless it can be verified to be secure. The chosen block cipher is Corrected Block TEA (XXTEA), which was developed by David Wheeler & Roger Needham at the Cambridge University Computer Lab. The algorithm is not patented. XXTEA is run in counter (CTR) mode to generate a stream of pseudorandom data. The counter mode is implemented by encrypting successive values of a basic counter (encrypting 1, then 2, then 3, etc...). The output stream is then XORed with data from `Math.random`. This last step guarantees that the randomness of our final numbers is at least as good as that of Math.random. (As proof of this, even if the initial pseudorandom data were so poor that it were all zeros, then the output would be the same as that of `Math.random`.)

## Usage ##

### Step 1 ###

Include the script.
```html
	<head>
		<script src="uuid.js"></script>
	</head>
```

### Step 2 ###

Initialize the generator by passing it an array of four random 32-bit unsigned integers integers (integers greater than or equal to 0 and less than 2^32) to use as the seed.
These numbers should be sent to the client from the server. If these are generated on the client using the browser's built-in `Math.random`, there is a greater likelihood that two browsers will end up generating the same four numbers because the browsers could have been seeded with the same timestamp.

```javascript
	// Replace the below numbers with your own random numbers generated on the server.
	var seed = [2752792635, 86914185, 58396346, 119515605];
	Uuid.initialize(seed);
```

### Step 3 ###

Create UUIDs.

```javascript
	Uuid.create(); // -> '96a3a1fe-df48-4b7a-b8aa-d3acfca9f74e'
```

## License ##

Copyright (c) 2012 Joshua Lawrence
Released under the MIT license.

XXTEA implementation copyright (c) Chris Veness 2002-2010: www.movable-type.co.uk/tea-block.html
Released under the Creative Commons Attribution 3.0 Unported License.

## Citations ##

[1] Klein, Amit, "Temporary user tracking in major browsers and Cross-domain information leakage and attacks",
http://www.trusteer.com/list-context/publications/temporary-user-tracking-major-browsers-and-cross-domain-information-leakag
September-November 2008
