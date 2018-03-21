# Introduction

Planning document for refactoring the R module in SWIG - the
simplified wrapper interface generator. The purpose of this document,
which doubles as a report for phase 1 of the project, is to catalog
the inappropriate use of string comparisons in the R module. The
next phase will replace these occurences with the appropriate use of the
swig parse tree.

# Background

SWIG generates C/C++ code and interpreter code to interface between an
interpreted language, such as R, and C/C++ code. Each target language
supported by SWIG has a swig C++ module providing the required
language specific functions. There is also an extensive set of typemap
configuration files that provide default behaviour that can be overridden
on a per-project basis. This project is refactoring the C++ module.

# Report structure

This report is a catalog of problem areas, which will be displayed as
hyperlinks into a github hosted SWIG repository.

