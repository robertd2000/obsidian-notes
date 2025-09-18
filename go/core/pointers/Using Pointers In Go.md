https://www.ardanlabs.com/blog/2014/12/using-pointers-in-go.html

Introduction  
I am asked quite a bit about when and when not to use pointers in Go. The problem most people have, is that they try to make this decision based on what they think the performance tradeoff will be. Hence the problem, don’t make coding decisions based on unfounded thoughts you may have about performance. Make coding decisions based on the code being idiomatic, simple, readable and reasonable.

  
  
My use of pointers is based on discoveries I have made looking at code from the standard library. There are always exceptions to these rules, but what I will show you is common practice. It starts with classifying the type of value that needs to be shared. These type classifications are built-in, struct and reference types. Let’s look at each one individually.  
  
Built-In Types  
Go’s [built-in types](http://golang.org/ref/spec#Types) represent primitive data values that are the building blocks for managing and working with data. I group these types collectively as the set of boolean, numeric and string types. When declaring functions and methods that accept values of these types, the standard library rarely shares them with a pointer.  
  
Let’s start by looking at the isShellSpecialVar function from the env package:  
  
  **Listing 1**    

[http://golang.org/src/os/env.go](http://golang.org/src/os/env.go)  
  
38 func isShellSpecialVar(c uint8) bool {  
39     switch c {  
40     case '*', '#', '$', '@', '!', '?', '0', '1', '2', '3', '4', '5', '6', '7', '8', '9':  
41         return true  
42     }  
43     return false  
44 }

  
The isShellSpecialVar function in listing 1 is declared to accept a value of type uint8 and return a value of type bool. For the caller to use this function, they must pass a copy of their uint8 type value into the function. This is the same for the return value. A copy of the function’s bool type value is being returned back to the caller.  
  
Next, let’s look at the getShellName function from the same env package:  
  
  **Listing 2**    

[http://golang.org/src/os/env.go](http://golang.org/src/os/env.go)  
  
54 func getShellName(s string) (string, int) {  
55     switch {  
56     case s[0] == '{':  
           . . .  
66         return "", 1 // Bad syntax; just eat the brace.  
67     case isShellSpecialVar(s[0]):  
68         return s[0:1], 1  
69     }  
       . . .  
74     return s[:i], i  
75 }

  
The getShellName function in listing 2 is declared to accept a value of type string and return two values, one of type string and the other of type int. A string is a special built-in type in Go that represents an immutable slice of bytes. Since this slice can’t grow, the capacity value is not associated with its [slice header](https://www.ardanlabs.com/blog/2013/08/understanding-slices-in-go-programming.html). It is best to treat values of type string the same way you treat boolean and numeric type values, as a primitive data value.  
  
When a call is made to getShellName, the caller passes a copy of its string value into the function. The function generates a new string value and returns a copy of that value back to the caller. All the values being passed in and out of this function are copies of the original values.  
  
This practice of sharing copies of string values is very prevalent in the strings package:  
  
  **Listing 3**    

[http://golang.org/src/strings/strings.go](http://golang.org/src/strings/strings.go)  
  
620 func Trim(s string, cutset string) string {  
621     if s == "" || cutset == "" {  
622         return s  
623     }  
624     return TrimFunc(s, makeCutsetFunc(cutset))  
625 }

  
All of the functions in the strings package accept copies of the caller’s string values and return copies of the string values they create. Listing 3 shows the implementation of the Trim function. The function accepts copies of two string values, and returns a copy of either the first string value that is passed in or a copy of a new string value that has trimmed out the cutset.  
  
If you review more code from the standard library that share built-in type values, you will see how these values are rarely shared with a pointer. If a function or method needs to change the value of a built-in type, a new value reflecting that change is often returned back to the caller.  
  
In general, don’t share built-in type values with a pointer.  
  
Struct Types  
[Struct](http://golang.org/ref/spec#Struct_types) types allow for the creation of complex data types by composing different types together. This is accomplished by composing a sequence of fields, each which with a name and a type. They also support [embedding](https://www.ardanlabs.com/blog/2014/05/methods-interfaces-and-embedded-types.html), which adds to the way struct types can be composed.  
  
Struct types can be implemented to behave like built-in types. When they are, you should treat them as such. To see a struct type that behaves as a primitive data value, we can look at the time package:  
  
  **Listing 4**    

[http://golang.org/src/time/time.go](http://golang.org/src/time/time.go)  
  
39 type Time struct {  
40     // sec gives the number of seconds elapsed since  
41     // January 1, year 1 00:00:00 UTC.  
42     sec int64  
43  
44     // nsec specifies a non-negative nanosecond  
45     // offset within the second named by Seconds.  
46     // It must be in the range [0, 999999999].  
47     nsec int32  
48  
49     // loc specifies the Location that should be used to  
50     // determine the minute, hour, month, day, and year  
51     // that correspond to this Time.  
52     // Only the zero Time has a nil Location.  
53     // In that case it is interpreted to mean UTC.  
54     loc *Location  
55 }

  
Listing 4 shows the Time struct type. This type represents time and has been implemented to behave as a primitive data value. If you look at the factory function Now, you will see it returns a value of type Time, not a pointer:  
  
  **Listing 5**    

[http://golang.org/src/time/time.go](http://golang.org/src/time/time.go)  
  
781 func Now() Time {  
782     sec, nsec := now()  
783     return Time{sec + unixToInternal, nsec, Local}  
784 }

  
Listing 5 shows how the Now function returns a value of type Time. This is an indication that values of type Time are safe to copy and is the preferred way to share them. Next, let’s look at a method that is used to change the value of a Time value:  
  
  **Listing 6**    

[http://golang.org/src/time/time.go](http://golang.org/src/time/time.go)  
  
610 func (t Time) Add(d Duration) Time {  
611     t.sec += int64(d / 1e9)  
612     nsec := int32(t.nsec) + int32(d%1e9)  
613     if nsec >= 1e9 {  
614         t.sec++  
615         nsec -= 1e9  
616     } else if nsec < 0 {  
617         t.sec--  
618         nsec += 1e9  
619     }  
620     t.nsec = nsec  
621     return t  
622 }

  
Just like we saw when working with built-in types, listing 6 shows how the Add method is called against a copy of the caller’s Time value. The method changes the local copy of the receiver value and returns a copy of that change back to the caller.  
  
Functions that accept Time values also accept copies of these values:  
  
  **Listing 7**    

[http://golang.org/src/time/time.go](http://golang.org/src/time/time.go)  
  
1118 func div(t Time, d Duration) (qmod2 int, r Duration) {

  
Listing 7 shows the declaration of the div function which accepts a value of type Time and Duration. Again, values of type Time are treated like a primitive data type and are copied when shared.  
  
Most of the time struct types are not created to behave like a primitive data type. In these cases, sharing the value by using a pointer is a better way to go. Let’s look at an example from the os package:  
  
  **Listing 8**    

[http://golang.org/src/os/file.go](http://golang.org/src/os/file.go)  
  
238 func Open(name string) (file *File, err error) {  
239     return OpenFile(name, O_RDONLY, 0)  
240 }

  
In listing 8 we see the Open function from the os package. It opens a file for reading and returns a pointer to a value of type File. Next, let’s look at the declaration of the File struct type for the UNIX platform:  
  
  **Listing 9**    

[http://golang.org/src/os/file_unix.go](http://golang.org/src/os/file_unix.go)  
  
15 // File represents an open file descriptor.  
16 type File struct {  
17     *file  
18 }  
19  
20 // file is the real representation of *File.  
21 // The extra level of indirection ensures that no clients of os  
22 // can overwrite this data, which could cause the finalizer  
23 // to close the wrong file descriptor.  
24 type file struct {  
25     fd int  
26     name string  
27     dirinfo *dirInfo // nil unless directory being read  
28     nepipe int32 // number of consecutive EPIPE in Write  
29 }

  
I left the comments for these type declarations in listing 9 because they really bring home the point I want to make. When you have a factory function like Open that is providing you a pointer, it is a good sign that you should not be making copies of the referenced value being returned. Open is returning a pointer because it is not safe to make copies of the referenced File value being returned. The value should always be used and shared through the pointer.  
  
Even if a function or method is not changing the state of a File struct type value, it still needs to be shared with a pointer. Let’s look at the epipecheck function from the os package for the UNIX platform:  
  
  **Listing 10**    

[http://golang.org/src/os/file_unix.go](http://golang.org/src/os/file_unix.go)  
  
58 func epipecheck(file *File, e error) {  
59     if e == syscall.EPIPE {  
60         if atomic.AddInt32(&file.nepipe, 1) >= 10 {  
61             sigpipe()  
62         }  
63     } else {  
64         atomic.StoreInt32(&file.nepipe, 0)  
65     }  
66 }

  
Here in listing 10, the epipecheck function accepts a pointer of type File. The caller therefore shares its File type value with the function via a pointer. Notice the epipecheck function is not changing the state of the File value but using it to perform its operation.  
  
This applies as well for the methods declared for the File type:  
  
  **Listing 11**    

[http://golang.org/src/os/file.go](http://golang.org/src/os/file.go)  
  
224 func (f *File) Chdir() error {  
225     if f == nil {  
226         return ErrInvalid  
227     }  
228     if e := syscall.Fchdir(f.fd); e != nil {  
229         return &PathError{"chdir", f.name, e}  
230     }  
231     return nil  
232 }

  
The Chdir method in listing 11 is using a pointer receiver to implement the method and does not change the state of the receiver value. In all these cases, to share a value of type File, it must be done with a pointer. A File value is not a primitive data value.  
  
If you review more code from the standard library, you will see how struct types are either implemented as a primitive data value like the built-in types or implemented as a value that needs to be shared with a pointer and never copied. The factory functions for a given struct type will give you a great clue as to how the type is implemented.  
  
In general, share struct type values with a pointer unless the struct type has been implemented to behave like a primitive data value.  
  
_If you are still not sure, this is another way to think about. Think of every struct as having a nature. If the nature of the struct is something that should not be changed, like a time, a color or a coordinate, then implement the struct as a primitive data value. If the nature of the struct is something that can be changed, even if it never is in your program, it is not a primitive data value and should be implemented to be shared with a pointer. Don’t create structs that have a duality of nature_.  
  
Reference Types  
Reference types are slices, maps, channels, interface and function values. These are values that contain a header value that references an underlying data structure via a pointer and other meta-data. We rarely share reference type values with a pointer because the header value is designed to be copied. The header value already contains a pointer which is sharing the underlying data structure for us by default.  
  
Let’s look at an example from the net package:  
  
  **Listing 12**    

[http://golang.org/src/net/ip.go](http://golang.org/src/net/ip.go)  
  
32 type IP []byte

  
Listing 12 shows a named type from the net package called IP with a base type that is a slice of bytes. There is value in using a named type when you need to declare behavior around a built-in or reference type. Let’s look at the MarshalText method for the IP named type:  
  
  **Listing 13**    

[http://golang.org/src/net/ip.go](http://golang.org/src/net/ip.go)  
  
329 func (ip IP) MarshalText() ([]byte, error) {  
330     if len(ip) == 0 {  
331         return []byte(""), nil  
332     }  
333     if len(ip) != IPv4len && len(ip) != IPv6len {  
334         return nil, errors.New("invalid IP address")  
335     }  
336     return []byte(ip.String()), nil  
337 }

  
Here in listing 13, we can see how the MarshalText method is using a value receiver. This is exactly what I would expect to see because we don’t share reference types with a pointer. If you look at the rest of the methods declared for the IP named type in the net package, you will see the use of more value receivers.  
  
This applies to sharing reference type values as parameters to functions and methods:  
  
  **Listing 14**    

[http://golang.org/src/net/ip.go](http://golang.org/src/net/ip.go)  
  
318 // ipEmptyString is like ip.String except that it returns  
319 // an empty string when ip is unset.  
320 func ipEmptyString(ip IP) string {  
321     if len(ip) == 0 {  
322         return ""  
323     }  
324     return ip.String()  
325 }

  
The ipEmptyString function in listing 14 accepts a value of named type IP. No pointer is used to share this value since the base type for IP is a slice of bytes and therefore a reference type.  
  
There is one common exception to the rule of not sharing a reference type with a pointer:  
  
  **Listing 15**    

[http://golang.org/src/net/ip.go](http://golang.org/src/net/ip.go)  
  
341 func (ip *IP) UnmarshalText(text []byte) error {  
342     if len(text) == 0 {  
343         *ip = nil  
344         return nil  
345     }  
346     s := string(text)  
347     x := ParseIP(s)  
348     if x == nil {  
349         return &ParseError{"IP address", s}  
350     }  
351     *ip = x  
352     return nil  
353 }

  
Anytime you are unmarshaling data into a reference type, you will need to share that reference type value with a pointer. Listing 15 shows the UnmarshalText method that is performing an unmarshal operation and is declared with a pointer receiver. The Decode and Unmarshal functions from the encoding packages would also expect to receive a pointer to a reference type.  
  
If you review more code from the standard library, you will see how values from reference types in most cases are not shared with a pointer. Since the reference type contains a header value whose purpose is to share an underlying data structure, sharing these values with a pointer is unnecessary. There is already a pointer in use.  
  
In general, don’t share reference type values with a pointer unless you are implementing an unmarshal type of functionality.  
  
Slices Of Values  
One thing I avoid when I can is storing data with a slice of pointers. When I retrieve data from a database, the web or even a file, I store that data in a slice of values:  
  
  **Listing 16**    

10 func FindRegion(s *Service, region string) ([]BuoyStation, error) {  
11     var bs []BuoyStation  
12     f := func(c *mgo.Collection) error {  
13         queryMap := bson.M{"region": region}  
14         return c.Find(queryMap).All(&bs)  
15     }  
16  
17     if err := s.DBAction(cfg.Database, "buoy_stations", f); err != nil {  
18         return nil, err  
19     }  
20  
21     return bs, nil  
22 }

  
Here is some code in listing 16 from one of my projects that makes a call to a MongoDB database via the mgo package. On line 14, I pass the address of the bs slice to the All method. The All method performs an unmarshal call underneath to create the values for the slice. Then the slice of data values is returned by passing a copy of the slice header value back to the caller.  
  
Using a slice of values allows the data for the program to be stored in a contiguous block of memory. This means that more of the core data I am using can be cached by the CPU at once and hopefully stays in the cache longer. If I create a slice of pointers, there is no guarantee the memory for these core data values would be contiguous, only the pointers to those values would be stored contiguously. Though I am thinking about performance in this case, I would argue that it is more idiomatic.  
  
There are times when this is not possible. Imagine if I needed a slice of File type values. Since I can’t make copies of File type values, I would need to create a slice of File type pointers. This situation occurs often when working with struct types from the standard library and not your own.  
  
In general, create slices and maps of values when you can.  
  
Conclusion  
The standard library is fairly consistent in how it shares values based on the type of value it is working with. Don’t use pointers with built-in data types unless you have a special need to do so. Struct types have a duality. If the struct type is implemented as a primitive data type, then don’t use a pointer. If not, then share those values with a pointer. Finally, reference types should not be shared with a pointer with very few exceptions.  
  
I would like to re-iterate three other thoughts as I close. First, make coding decisions based on the code being idiomatic, simple, readable and reasonable. Second, this is not about right and wrong, think about the code you are writing and there being reason behind the decisions you are making. Lastly, take each situation and scenario as an individual case, and try not to apply a blanket pattern or solution.