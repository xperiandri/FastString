# FastString: strings in a hurry

.NET's builtin strings are convenient. However, they are not designed with performance in mind.

FastString offers currently two methods for working with strings that can be significantly faster
than .NET's:


## StringView

Strings are immutable. Taking a substring does not need to allocate memory. However, System.String
copies over data when you take a substring.

StringView addresses that. It's a view into a slice of a string, supporting some of the same
operations. (In the long term, I aim to extend it to support the same interface as System.String.)

StringView is good for substrings that do not significantly outlive the string they came from. If
you extract out a small subset of a large string with StringView and then throw out that large
string, it's kept in memory until the StringView goes out of scope. Not so great for grabbing just a
couple values from a giant configuration file, for instance.

StringView is a struct to reduce allocations further.


## Utf8String

The world has largely settled on UTF8 as the standard string format. Why transcode to UTF16 and back
constantly?

The `Utf8String` struct aims for `System.String` parity, but using UTF8 as the encoding. Like StringView,
it allocates as little as possible.

Its allied types are:
* `Utf8Enumerator`: iterate through Unicode codepoints in a Utf8String.
* `ReverseUtf8Enumerator`: iterate through Unicode codepoints in a Utf8String, backwards.
* `Utf8Builder`: like StringBuilder, but for UTF8.
* `Utf8Writer`: like StringBuilder, but for UTF8 and writing to a Stream that you provide.

The **major caveat** when working with `Utf8String`: if you forge a `Utf8String` instance from a mutable byte
buffer, you **must** clone it before altering the buffer. Since it doesn't copy the data by default,
changes in the underlying buffer are changes in the string!


## Real-world results

The test project is a compiler that turns a markdown-ish format into epub documents. When using
System.String to compile the 413KB reference document, run time took 1.02 seconds and allocated
3.8GB of String objects.

When using StringView, the run time was reduced to 0.12 seconds and allocations were reduced to
3.8MB of String objects. Other allocations were not significantly changed.

When using Utf8String, the run time was reduced to 0.03 seconds. String allocations were reduced to 300KB.
Other allocations were slightly reduced: `System.Char[]` allocations were converted to
`System.Byte[]` and dropped by 300KB.
