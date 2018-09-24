# Introduction

Developments in the master branch of swig have shifted priorities somewhat,
with changes to the enumeration framework resulting in R code generated
for the SimpleITK project failing to parse. The implementation phase
of the project has thus addressed two problems:

1. Enumeration framework - enabling support for complex enumeration structures
   using a similar strategy to other language modules.
   
1. Refactoring of the accessor framework, which provides ability to
   read or components variables in structures or classes.
   
## Enumerations

This part of the project is implemented in [PR 1328](https://github.com/swig/swig/pull/1328)

### Problems

The R module attempts to perform some basic guesswork as to the
integer values underlying the enumeration names. This approach becomes
unreliable as soon as the typical compiler defaults are bypassed. Some recent
developments have attempted to directly import the C code to R, which works
for simple expressions, but results in syntactically incorrect R code in others.

SimpleITK uses initialization code like this:

```cpp
enum PixelIDValueEnum {
  sitkUnknown = -1,
  sitkUInt8 = PixelIDToPixelIDValue< BasicPixelID<uint8_t> >::Result,   ///< Unsigned 8 bit integer
  sitkInt8 = PixelIDToPixelIDValue< BasicPixelID<int8_t> >::Result,     ///< Signed 8 bit integer
  sitkUInt16 = PixelIDToPixelIDValue< BasicPixelID<uint16_t> >::Result, ///< Unsigned 16 bit integer
  sitkInt16 = PixelIDToPixelIDValue< BasicPixelID<int16_t> >::Result,   ///< Signed 16 bit integer
  sitkUInt32 = PixelIDToPixelIDValue< BasicPixelID<uint32_t> >::Result, ///< Unsigned 32 bit integer
  sitkInt32 = PixelIDToPixelIDValue< BasicPixelID<int32_t> >::Result,   ///< Signed 32 bit integer
  sitkUInt64 = PixelIDToPixelIDValue< BasicPixelID<uint64_t> >::Result, ///< Unsigned 64 bit integer

```

which leads to syntactically incorrect R code.

### Approach

The approach used by other swig language modules is to create
accessors for C/C++ representions of the enumeration and use those
to retrieve underlying the integer representations. This is the only
reliable way, and requires run-time support rather than deducing the
values at binding generation time.

### Enumerations in R

R doesn't have a native enumeration type. The swig module implements
an enumeration framework using strings and special lookup
functions. The lookup functions store a vector of values in a hidden
environment for each enumeration. The type of each enumeration is
recorded as an attribute for functions using it as an input or output,
and this record is used access the correct environment as well as to
support overloading of methods/functions.

The content of enumeration environments is initialised via calls to
a _defineEnumeration_ function.

### Issues

There were some issues to overcome using this approach for the R
module.

The most important issue is the requirement of _defineEnumeration_ to
access the shared library during package load time. The code generated
by swig would typically be placed in a single file in the package/R
folder and would include all necessary calls to
_defineEnumeration_. This approach was acceptable when default values
for integers underlying the enumerations were supplied at binding
generation time. However the new approach requires calls to the
compiled C/C++ code in order to determine the correct values. The
library containing these calls is not available until the end of the
package loading sequence, resulting in errors from _defineEnumeration_ calls.

This issue has been avoided by using R _promises_ inside
_defineEnumeration_, so that calls to the shared library are not made
until the enumeration is used.

The proposed implementation passes the SimpleITK tests.

### Limitations and future work

The main issue to sort out relates to name mangling. The current
R/swig module mangles namespaces into code at various points. These
are not generally user visible names, and are mostly related to types
used to support dispatching and internal S4 class IDs. I am awaiting
feedback from the head swig maintainer before attempting to address
this issue further.

The main problem appears to be that some languages supported by swig
have their own notion of namespaces that can be sensibly be used to
approximate C++ namespaces, and the binding generation should be aware
of it. R doesn't have this, and the use of namespaces in the various
type tags offers a level of saftey that I think is worthwhile.

## Accessors

This part of the project is implemented in [PR 1268](https://github.com/swig/swig/pull/1268)

### Problems

The accessor framework creates functions to set or retrieve values for
variables defined in a namespace or contained in a class or structure. For example, code like:

```cpp
// bool variables
bool bool1 = true;

```

leads to R functions, _bool1\_set_ and _bool1\_get_, as well as C
functions _R\_swig\_bool1\_set_ and _R\_swig\_bool1\_get_. 

The R module used logic based around the names of accessor functions
to create the necessary binding code. There were bugs in that code
and, more seriously, the logic lead to problems if there were already
functions with names matching the patterns being tested for. For
example, library functions ending in _\_set_ would not get bound
correctly because they were interpreted as swig generated wrappers.

This section of the code has been rewritten to use information in the
swig parse tree, which includes flags indicating whether a function is
a "memberset" or "memberget" type (i.e autogenerated by swig).

The resulting code is longer, but deals correctly with many previous
problem cases.

There were also some refactoring to use services provided by the core
swig infrastructure, such as registration of appropriate wrapper
names, rather than replacing later and use of built in calls to return
names of accessors.
