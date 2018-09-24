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

This part of the project is implemented in [PR 1328](https://github.com/swig/swig#1328)


