---
title: "Open Projects"
layout: textlay
excerpt: "Projects"
sitemap: false
permalink: /open_projects
---

# Open Projects

<nav>
  <h4>Table of Contents</h4>
  * this unordered seed list will be replaced by toc as unordered list
  {:toc}
</nav>


# Add initial integration of Clad with Enzyme

## Description

In mathematics and computer algebra, automatic differentiation (AD) is a set of
techniques to numerically evaluate the derivative of a function specified by a
computer program. Automatic differentiation is an alternative technique to
Symbolic differentiation and Numerical differentiation (the method of finite
differences). Clad is based on Clang which provides the necessary facilities for
code transformation. The AD library is able to differentiate non-trivial
functions, to find a partial derivative for trivial cases and has good unit test
coverage. Enzyme is a prominent autodiff framework which works on LLVM IR.

Clad and Enzyme can be considered as a C++ frontend and a backend automatic
differentiation framework. In many cases, when clad needs to fall back to
numeric differentiation it can try configuring and using Enzyme to perform the
automatic differentiation on lower level.

## Task ideas and expected results

Understand how both systems work. Define the Enzyme configuration requirements
and enable Clad to communicate efficiently with Enzyme. That may require several
steps: start building and using the optimization pass of Enzyme as part of the
Clad toolchain; use Enzyme for cross-validation derivative results; etc.


# Add numerical differentiation support in clad

## Description

In mathematics and computer algebra, automatic differentiation (AD) is a set of
techniques to numerically evaluate the derivative of a function specified by a
computer program. Automatic differentiation is an alternative technique to
Symbolic differentiation and Numerical differentiation (the method of finite
differences). Clad is based on Clang which provides the necessary facilities for
code transformation. The AD library is able to differentiate non-trivial
functions, to find a partial derivative for trivial cases and has good unit test
coverage. In a number of cases, due different limitations, it is either
inefficient or impossible to differentiate a function.

Currently, clad cannot differentiate declared-but-not-defined functions.
In that case it issues an error. Instead, clad should fall back to its future
numerical differentiation facilities.

## Task ideas and expected results

Implement numerical differentiation support in clad. It should be available
through a dedicated interface (for example `clad::num_differentiate`).
The new functionality should be connected to the forward mode automatic
differentiation. If time permits, a prototype of configurable error estimation
for the numerical differentiation should be implemented.


# Add support for functor objects in clad

## Description

In mathematics and computer algebra, automatic differentiation (AD) is a set of
techniques to numerically evaluate the derivative of a function specified by a
computer program. Automatic differentiation is an alternative technique to
Symbolic differentiation and Numerical differentiation (the method of finite
differences). Clad is based on Clang which provides the necessary facilities for
code transformation. The AD library is able to differentiate non-trivial
functions, to find a partial derivative for trivial cases and has good unit test
coverage.

Many computations are modelled using functor objects. Usually, a functor object
is a lightweight C++ object which has state stored as members and has overridden
call operator (`operator()`). For example:

```cpp
struct Functor {
  double x;
  double operator()() { return x * x ;}
};

int main () {
  Functor Fn;
  Fn.x = 2;
  double pow2 = Fn();
  auto dpow2_dx = clad::differentiate(Fn, /*wrt*/ 0); // unsupported
  return pow2;
}
```

The goal of this project is to modify Clad to handle such cases.


## Task ideas and expected results

Implement functor object differentiation in both forward and reverse mode. The
candidate should be ready to investigate performance bottlenecks, add test and
benchmarking coverage and improve documentation for various parts of clad not
only limited to the functor object differentiation support.



# Utilize second order derivatives from Clad in ROOT

## Description

In mathematics and computer algebra, automatic differentiation (AD) is a set of
techniques to numerically evaluate the derivative of a function specified by a
computer program. Automatic differentiation is an alternative technique to
Symbolic differentiation and Numerical differentiation (the method of finite
differences). Clad is based on Clang which provides the necessary facilities for
code transformation.

ROOT is a framework for data processing, born at CERN, at the heart of the
research on high-energy physics. Every day, thousands of physicists use ROOT
applications to analyze their data or to perform simulations. ROOT has a
clang-based C++ interpreter Cling and integrates with Clad to enable flexible
automatic differentiation facility.

## Task ideas and expected results

TFormula is a ROOT class which bridges compiled and interpreted code. Teach
TFormula to use second order derivatives (using `clad::hessian`). The
implementation should be very similar to what was done in
[this pull request](https://github.com/root-project/root/pull/2745).
The produced code should be well tested and documented. If time permits, we
should pursue converting the C++ gradient function to CUDA device kernels.
The integration of that feature should be comparable in terms of complexity to
integrating `clad::hessians`.


# Improve Cling's Development Lifecycle

## Description

Cling is an interactive C++ interpreter, built on top of Clang and LLVM compiler
infrastructure. Cling realizes the read-eval-print loop (REPL) concept, in order
to leverage rapid application development. Implemented as a small extension to
LLVM and Clang, the interpreter reuses their strengths such as the praised
concise and expressive compiler diagnostics.

## Task ideas and expected results

The project foresees to enhance the Github Actions infrastructure by adding
development process automation tools:
  * Code Coverage information (`codecov`)
  * Static code analysis (`clang-tidy`)
  * Coding conventions checks (`clang-format`)
  * Release binary upload automation

# Allow redefinition of CUDA functions in Cling

## Description

Cling is an interactive C++ interpreter, built on top of Clang and LLVM compiler
infrastructure. Cling realizes the read-eval-print loop (REPL) concept, in order
to leverage rapid application development. Implemented as a small extension to
LLVM and Clang, the interpreter reuses their strengths such as the praised
concise and expressive compiler diagnostics.

Since the development of Cling started, it got some new features to enable new
workflows. One of the features is CUDA mode, which allows you to interactively
develop and run CUDA code on Nvidia GPUs. Another feature is the redefinition of
functions, variable classes and more, bypassing the one-definition rule of C++.
This feature enables comfortable rapid prototyping in C++. Currently, the two
features cannot be used together because parsing and executing CUDA code behaves
differently compared to pure C++.

## Task ideas and expected results

The task is to adapt the redefinitions feature of the pure C++ mode for the CUDA
mode. To do this, the student must develop solutions to known and unknown
problems that parsing and executing CUDA code causes.


# Improving Cling Reflection for Scripting Languages

## Description

Cling has basic facilities to make queries about the C++ code that it has
seen/collected so far. These lookups assume, however, that the caller knows what
it is looking for and the information returned, although exact, usually only
makes sense within C++ and is thus often too specific to be used as-is.

A scripting language, such as Python, that wants to make use of such lookups by
name, is forced to loop over all possible entities (classes, functions,
templates, enums, ...) to find a match. This is inefficient. Furthermore, many
lookups will be multi-stage: a function, but which overload? A template, but
which instantiation? A typedef, of what? The current mechanism forces the
scripting language to provide a type-based match, even where C++ makes
distinctions (e.g. pointer v.s. reference) that do not exist in the scripting
language. This, too, makes lookups very inefficient.

The returned information, once a match is found, is exact, but because of its
specificity, requires the caller to figure out C++ concepts that have no meaning
in the scripting language. E.g., there is no reason for Python to consider an
implicitly instantiated function template different from an explicitly
instantiated one.

## Task ideas and expected results

The project should start with a design C++ entities grouping that make sense
for scripting languages and design a query API on top of these. Special
consideration should be given to the various name aliasing mechanisms in C++
(e.g., `using` and `typedef`), especially as relates to C++ access rules.
This design can likely start from the existing Cling wrapper API from Cppyy
and modify it to improve consistency, remove redundancy, and enable usage
patterns that minimize lookups into Cling.

Next is a redesign of Cling's lookup facilities to support this new API
efficiently, with particular care to use cases that require multiple,
consecutive/related, lookups, such as finding a specific function template
instantiation, or step-wise resolution of a typedef.

This design is then to be implemented, using test-driven development. As a
stretch goal, the cppyy-backend should be modified to use the new API.

# Developing C++ modules support in CMSSW and Boost

## Description

The LHC smashes groups of protons together at close to the speed of light: 40
million times per second and with seven times the energy of the most powerful
accelerators built up to now. Many of these will just be glancing blows but some
will be head on collisions and very energetic. When this happens some of the
energy of the collision is turned into mass and previously unobserved,
short-lived particles – which could give clues about how Nature behaves at a
fundamental level - fly out and into the detector. Our work includes the
experimental discovery of the Higgs boson, which leads to the award of a Nobel
prize for the underlying theory that predicted the Higgs boson as an important
piece of the standard model theory of particle physics.

CMS is a particle detector that is designed to see a wide range of particles and
phenomena produced in high-energy collisions in the LHC. Like a cylindrical
onion, different layers of detectors measure the different particles, and use
this key data to build up a picture of events at the heart of the collision.

Last year, thanks to [Lucas Calmolezi and GSoC](https://summerofcode.withgoogle.com/archive/2020/projects/5397144158076928/),
the usage of boost in CMSSW was modernized. It improved the C++ modules support
of local boost fork.

## Task ideas and expected results

Many of the accumulated local patches add missing includes to the relevant boost
header files. The candidate should start by proposing the existing patches to
the boost community. Try to compile more boost-specific modules which is mostly
a mechanical task. The student should be ready to work towards making the C++
module files more efficient containing less duplications. The student should be
prepared to write a progress report and present the results.


# Implement a shared-memory based JITLinkMemoryManager for out-of-process JITting

## Description

Write a shared-memory based JITLinkMemoryManager.

LLVM’s JIT uses the JITLinkMemoryManager interface to allocate both working
memory (where the JIT fixes up the relocatable objects produced by the compiler)
and target memory (where the JIT’d code will reside in the target).
JITLinkMemoryManager instances are also responsible for transporting fixed-up
code from working memory to target memory. LLVM has an existing cross-process
allocator that uses remote procedure calls (RPC) to allocate and copy bytes to
the target process, however a more attractive solution (when the JIT and target
process share the same physical memory) would be to use shared memory pages to
avoid copies between processes.

## Task ideas and expected results:

Implement a shared-memory based JITLinkMemoryManager:
  * Write generic LLVM APIs for shared memory allocation.
  * Write a JITLinkMemoryManager that uses these generic APIs to allocate shared
  working-and-target memory.
  * Make an extensive performance study of the approach.


# Modernize the LLVM "Building A JIT" tutorial series

## Description

The LLVM BuildingAJIT tutorial series teaches readers to build their own JIT
class from scratch using LLVM’s ORC APIs, however the tutorial chapters have not
kept pace with recent API improvements. Bring the existing tutorial chapters up
to speed, write up a new chapter on lazy compilation (chapter code already
available) or write a new chapter from scratch.

## Task ideas and expected results:

* Update chapter text for Chapters 1-3 -- Easy, but offers a chance to get
up-to-speed on the APIs.
* Write chapter text for Chapter 4 -- Chapter code is already available, but no
chapter text exists yet.
* Write a new chapter from scratch -- E.g. How to write an out-of-process JIT,
or how to directly manipulate the JIT'd instruction stream using the
ObjectLinkingLayer::Plugin API.

# Write JITLink support for a new format/architecture

## Description

JITLink is LLVM’s new JIT linker API -- the low-level API that transforms
compiler output (relocatable object files) into ready-to-execute bytes in
memory. To do this JITLink’s generic linker algorithm needs to be specialized to
support the target object format (COFF, ELF, MachO), and architecture (arm,
arm64, i386, x86-64). LLVM already has mature implementations of JITLink for
MachO/arm64 and MachO/x86-64, and a relatively new implementation for
ELF/x86-64. Write a JITLink implementation for a missing target that interests
you. If you choose to implement support for a new architecture using the ELF or
MachO formats then you will be able to re-use the existing generic code for
these formats. If you want to implement support for a new target using the COFF
format then you will need to write both the generic COFF support code and the
architecture support code for your chosen architecture.

## Task ideas and expected results:

Write a JITLink specialization for a not-yet-supported format/architecture.


# Extend clang AST to provide information for the type as written in template instantiations.

## Description

When instantiating a template, the template arguments are canonicalized before
being substituted into the template pattern. Clang does not preserve type sugar
when subsequently accessing members of the instantiation.

```cpp
std::vector<std::string> vs;
int n = vs.front(); // bad diagnostic: [...] aka 'std::basic_string<char>' [...]

template<typename T> struct Id { typedef T type; };
Id<size_t>::type // just 'unsigned long', 'size_t' sugar has been lost
```
Clang should "re-sugar" the type when performing member access on a class
template specialization, based on the type sugar of the accessed specialization.
The type of vs.front() should be std::string, not std::basic_string<char, [...]>.

Suggested design approach: add a new type node to represent template argument
sugar, and implicitly create an instance of this node whenever a member of a
class template specialization is accessed. When performing a single-step desugar
of this node, lazily create the desugared representation by propagating the
sugared template arguments onto inner type nodes (and in particular, replacing
Subst*Parm nodes with the corresponding sugar). When printing the type for
diagnostic purposes, use the annotated type sugar to print the type as
originally written.

For good results, template argument deduction will also need to be able to
deduce type sugar (and reconcile cases where the same type is deduced twice with
different sugar).

## Task ideas and expected results:

Diagnostics preserve type sugar even when accessing members of a template
specialization. `T<unsigned long>` and `T<size_t>` are still the same type and
the same template instantiation, but `T<unsigned long>::type` single-step
desugars to 'unsigned long' and `T<size_t>::type` single-step desugars to
'size_t'.


# Infrastructure

## Improve Cling's packaging system cpt

Cling has a flexible tool which can build and package binaries. It is
implemented in python.

There are several improvements that can be made to cpt:
* Fix deb package creation
* Rewrite parts of cpt 
    * Use a `if __name__ == "__main__"` block as program execution starting point
    * No mutating global variables
    * Minimize use of `subprocess`
    * Making cpt flake8 compliant (flexible error/violation codes)
    * Revamp argument parser (Examine possibility of dependent arguments)

