https://dave.cheney.net/2014/03/25/the-empty-struct#comment-2815

It’s not true that “a value must be aligned in memory to a multiple of its width.” Each type has another property, its alignment. Alignments are always powers of two. The alignment of a basic type is usually equal to its width, but the alignment of a struct is the maximum alignment of any field, and the alignment of an array is the alignment of the array element. The maximum alignment of any value is therefore the maximum alignment of any basic type. Even on 32-bit systems this is often 8 bytes, because atomic operations on 64-bit values typically require 64-bit alignment.

To be concrete, a struct containing 3 int32 fields has alignment 4 but width 12.

It is true that a value’s width is always a multiple of its alignment. One implication is that there is no padding between array elements.