Slices are common place in Go code, but they can be a stubborn topic as they are Go's "replacement" for dealing with arrays. Except, slices are not arrays. The relationship between slices and arrays is a curious one ...

Here is some pseudo-Go code attempt at succinctly representing the basic relationship between a "slice" object and an "array" object:

```go
// A Slice is a mutable structure
// see there full expression here: https://golang.org/ref/spec#Slice_expressions
// we do not address the "max" argument of slices
type Slice struct{
    Ref *Element // pointer to smallest index in the accessible segment of the array
    Type string // immutable -- must match the Type of the array that it references
    Length int // length of accessible segment of indexes
    Capacity int // size of underlying array
}

// Arrays are structurally immutable
// Only values in the array are mutable
// The size of the array in memory is the cardinality * element_type_size
type Array struct{
    Cardinality int // immutable -- number of elements
    Type string // immutable -- element type
    Location [1]uintptr //immutable -- address of first element (index 0)
}
```

So, the basic relationship between a slice and an array is: slices are references to an array. The catch is: since slices are structurally mutable we can actually change *which* array a slice is referencing. We can mutate how much of an array a Slice references. But, the one thing we cannot mutate (because Go is statically-typed) is the Type of the arrays that a slice is allowed to reference.

## Neat Slice Tricks

We can manipulate slices with built-in functions in Go. Take for example: `append`.

Append is weird. Very weird. Check out some examples:

- [https://play.golang.org/p/LxdGYAJhEV](https://play.golang.org/p/LxdGYAJhEV)
- [https://play.golang.org/p/ZjDBuFTREK](https://play.golang.org/p/ZjDBuFTREK)
- [https://play.golang.org/p/578ujt2-Bp](https://play.golang.org/p/578ujt2-Bp)

Roughly: 

- `append` increases the length for a slice by 1 
- if the number of arguments (besides the initial slice arg) is greater than the capacity of the supplied slice then `append` will allocate a new array
- if the number of arguments (besides the initial slice arg) are equal to or less than the capacity, values in the array are simply overwritten starting with the first element in the slice reference
- `append` is modulo capacity: 
    + `append(slice, value) -> newSlice[(cap(slice) - len(slice)) % cap(slice)] = value, iff (cap(slice) - len(slice)) >= 0, else allocate new array`

We can find plenty of other neat tricks for manipulating slices using the built-in `append` and `copy` functions:

- here: [https://github.com/golang/go/wiki/SliceTricks](https://github.com/golang/go/wiki/SliceTricks)
- and here: [https://blog.golang.org/slices](https://blog.golang.org/slices)

But, if you are like me ... something just doesn't feel quite right about using "neat hacks, bruh" to manipulate slices, think in terms of, write and read code with, etc.

Perhaps, we can find a slightly more structured way to think of slices and arrays in Go?

## The Combinatorics of Slices

Unsurprisingly, there is also a combinatoric relationship between slices and arrays defined by the number of total possible (and valid) slices that can be defined over an array (of capacity n).

What we find is that for an array of capacity n, the total number of slices is equal to the binomial coefficient `(n+1 choose 2)`. In other words, it is always equal to a [triangular number](https://en.wikipedia.org/wiki/Triangular_number).

Here are the first six triangular numbers represented by pictures (taken from Wikipedia).

![](https://upload.wikimedia.org/wikipedia/commons/1/1c/First_six_triangular_numbers.svg)

We can use the above image of triangles to make sense of why triangular numbers are directly related to the total number of possible slices over an array: 

- Each triangle represents the array of capacity n (where n is the number of nodes in the bottom row of the triangle)
- Each node in a triangle represents a single possible valid slice over that array.
- The top node in the triangle represents the slice that has access to all elements in the array
- The bottom nodes in the triangle represent slices with access to only one element of the array.
- All slices in the triangle have the same capacity (defined by the number of nodes in the bottom row)
- Each level (height of row: starting with level of bottom row equal to 1) of the triangle represents the length of each slice in that level.

Why is this interesting? Well it was to me because it shows the structural relationship between all valid slices over an array in a single and simple diagram. 

Can these relationships between slices be defined further?

## The Algebra of Slices

Previously we talked about "neat tricks" that *can* be useful using `append` and `copy`. Instead of relying on these tricks alone let's see if we can use our combinatoric information and devise a more algebraic way of working with slices.

### Ring of Arrays (of Type T) `[c ∈ Cap]T{}`

First, we can define the `Ring of Arrays (of Type T)` as being based on the capacity of the array. This makes the ring isomorphic to the ring of Integers.

- [Axioms](https://en.wikipedia.org/wiki/Ring_(mathematics)#Definition) for a ring of arrays R.
    + (Addition of Capacities) ∀ a,b ∈ R: (a + b) ∈ R, this is closure under addition
    + (Commutative) ∀a,b ∈ R: a + b = b + a
    + (Associative) ∀a,b,c ∈ R: a + (b + c) = (a + b) + c
    + (Identity) ∃ id ∈ R → ∀a ∈ R: a + id = a, this is the 0 capacity
    + (Inverses) ∀a ∈ R: ∃ -a ∈ R → a + -a = id
    + (Multiplication of Capacities) ∀ a,b ∈ R: (a * b) ∈ R, this is closure under multiplication
    + Multiplication is associative
    + Multiplication has an identity: capacity 1
    + Multiplication is distributive (left and right) with relation to addition

All of the above axioms hold up for our Ring of Arrays (of Type T) as long as the addition and multiplication are defined for (performed on) the capacity of the array only.

The ignored story here is that this allows arrays of negative capacity to exist. But, we just consider those to be imaginary objects that simply behave as inverses to real array objects. In other words, a negative capacity array can never be instantiated but it can behave on a positive capacity array via addition or multiplication.

- Correlative Axiom: if an instantiated array (after addition or multiplication) is determined to have a capacity less than zero, then it simply becomes a zero array. 
    + Mathematically, all negative capacities with absolute value greater than or equal to the capacity of an array are inverses of that array under addition. 
    + And all negative capacities map any array to the zero capacity under multiplication.

Since arrays are structurally immutable: this array algebra is only good for determining the capacities of newly allocated arrays based on two or more other arrays.

This ring of arrays also does not tell us anything about slices and is therefore not very useful in of itself for Go.

### Slice Groups

We can consider the set of all slices over an array of capacity n to be an abelian group under a specific "composition" operation which we will define below via examples.

This means that each possible triangle (like the ones from the image above) represents a single possible slice group.

- [Axioms](https://en.wikipedia.org/wiki/Abelian_group#Definition)
    + (Inverse) ∀a ∈ A: a ∘ a = ϕ (every slice is its own inverse)
    + (Commutativity) ∀a,b ∈ A: a ∘ b = b ∘ a
    + (Associativity) ∀a,b,c ∈ A: (a ∘ b) ∘ c = a ∘ (b ∘ c)
    + (Identity) ϕ is the empty slice and is the identity
    + (Closure) ∀a,b ∈ A: a ∘ b ∈ A

The operation that the above axioms hold under is the `Symmetric Difference of Indexes` (which is the `XOR` operation on indexes). This operation is defined [here](https://en.wikipedia.org/wiki/Symmetric_difference) for sets. It behaves the same for slices.

For example, if one slice addresses indexes `{0, 1, 2}` and we take the symmetric difference between it and a slice that addresses index `{2}` of the same array then the resulting slice will be a slice that addresses indexes `{0,1}`.

Let's look at the following example for further clarification (here we are using a diagram like the ones from the triangle image above):

![](https://i.imgur.com/uMCd4Fl.png)

In the above abelian group of slices, the following statements are true:

- x ∘ f = y
- y ∘ f = x
- x ∘ h = g
- f ∘ h = z
- f ∘ g = h
- h ∘ h = ϕ
- x ∘ g = h
- etc.

Aha, now this seems a little more useful to us. But there is still one more thing we can discuss to make the picture of working with slices even clearer.

## Slice Modules

So, now that we have an algebra for arrays and an algebra for slices over an array of capacity n ... what happens when we combine these two concepts? :thinking:

Simple: we get modules of slices over the ring of arrays. The wot over the who?

Modules! The generalization of vector spaces defined over an algebraic field.

- [Axioms](https://en.wikipedia.org/wiki/Module_(mathematics)#Formal_definition): given a ring of arrays R and an abelian group of slices (M, ∘) with r,s ∈ R and x,y ∈ M, as well an operation ⋅ :R * M → M ...
    + r ⋅ (x ∘ y) = (r ⋅ x) ∘ (r ⋅ y)
    + (r + s) ⋅ x = (r ⋅ x) ∘ (s ⋅ x) 
    + (r * s) ⋅ x = r ⋅ (s ⋅ x)
    + 1ᵣ ⋅ x = x (multiplicative identity on the ring)

Each abelian group over an array of capacity n represents one possible Module (bimodule) over the Ring of Arrays (of Type T).

Except, there is a catch. Previously, we defined the abelian group of modules as being the set of all valid slices of an array of capacity n. 

However, the elements of a module (as an abelian group) are all valid slices from a single slice triangle but those slices can have *any* capacity.

We illustrate this in the following image:

![](https://i.imgur.com/MD9XoHb.png)

This means that our symmetric differentiation alone will not suffice. Instead we give it a simple modification: the composition operation in our module is defined by combining the capacities of the two slices and then doing symmetric differentiation on their indexes.

> (r₁ ⋅ x) ∘ (r₂ ⋅ y) = (r₁ + r₂) ⋅ (x ∘ y)
> 
> where r₁, r₂ ∈ R (are capacities), and x, y ∈ M (are slices in our module)

But, how can we verify now that two slices are on our module. Well, each module must have something called: a `base`. The `base` is the maximal length (length of the segment) a slice in our module is allowed to have.

Thus, we say that the `base` of a module is equivalent to the capacity (length of bottom row) of the slice triangle that is it's "basis". This determines the maximum length our slice is allowed to have to be considered a part of our module.

### Submodules & Module Homomorphisms

Each slice triangle is thus the "basis" for an R-Module. Which means that every module can have [sub-modules](https://en.wikipedia.org/wiki/Module_(mathematics)#Submodules_and_homomorphisms) which are represented by the sub-triangles of each slice triangle.

A (sub-)module homomorphism is a function `f: x ∈ M₁ → y ∈ M₂`. An example module homomorphism (where r₁ is the capacity of the slice):

```
f: M₁ → M₂

f: {              {
    r₁              r₁ + 1
    {indexes}  →    {indexes} ∘ {2,3,4}
    base₁           base₂
   }              }

Homomorphism is valid as long as: base(M₂) >= 5
```

another example:

```
g: M₁ → M₂

g: {              {
    r₁               r₁ + 1
    {n}     →        {n+1}
    base₁            base₂
   }              }

Homomorphism is valid as long as: base(M₂) >= base(M₁) + 1

```

But, it's often the case that you will not care about M₂ and it will simply hold true that the result of `f` or `g` on some element of M₁ will always be within some other module M₂. In other words: it will most likely be the case that we do not have to worry about the bases of our modules.

A homomorphism must simply preserve the structure. And as long as it does then it is validated.

Structure-Preservation Test of a homomorphism `f`:

-  `f(r ⋅ m ∘ s ⋅ n) = r ⋅ f(m) ∘ s ⋅ f(n)`

As long as the above holds then you are good to go.

## Example

Probably, the most common use case for slices in Go, is to have an object that acts as a list that we can arbitrarily grow over time.

Just using the `append` operation on an allocated array (i.e. allocating a new array with capacity++) is not the most performant way of accomplishing this. Thus, was born the idea of: *Dynamic Allocation* (long before Go or `append` existed).

Btw, you should be using dynamic allocation in your code as well! (and I should be too ... [woops](https://github.com/JKhawaja/fabric/blob/master/dg.go#L94)).

Examples of dynamic allocation functions for arrays in Go can be found: [here](https://blog.golang.org/slices) and [here](http://openmymind.net/Controlling-Array-Growth-In-Golang/). These dynamic allocation functions are used to *replace* the use of `append` in our code (in respect to any slice that we want to arbitrarily grow in a performant way).

Just in case you didn't feel like clicking through to those links and reading those articles as well as this one, here is some of the code:

```go
// Source: https://blog.golang.org/slices
func Extend(slice []int, element int) []int {
    n := len(slice)
    if n == cap(slice) {
        // Slice is full; must grow.
        // We double its size and add 1, so if the size is zero we still grow.
        newSlice := make([]int, len(slice), 2*len(slice)+1)
        copy(newSlice, slice)
        slice = newSlice
    }
    // increase length of slice by 1
    slice = slice[0 : n+1]
    // "append" new element to slice
    slice[n] = element
    return slice
}
```

What, we want to do now is rewrite a dynamic allocation procedure *in terms of our algebra of slices*.

- Run the example code here: [https://play.golang.org/p/4NzvJnFO2m](https://play.golang.org/p/4NzvJnFO2m)
- See the example code here: [https://gist.github.com/JKhawaja/d3fa5ca91ef0e41df3c5a3e4b634e913](https://gist.github.com/JKhawaja/d3fa5ca91ef0e41df3c5a3e4b634e913)

You can see our new `Extend` method (function) on [line 163](https://gist.github.com/JKhawaja/d3fa5ca91ef0e41df3c5a3e4b634e913#file-slice_algebra-go-L163).

## Interesting sidepoint: Arrays as Vectors

An array can be a vector if it is an array of Type {[Algebraic_Field](https://en.wikipedia.org/wiki/Field_(mathematics))}. This means that the top node in the slice triangle represents the Vector space of dimension n. And each node below that in the triangle represents the Vector space of a smaller dimension, where the bottom n-nodes represent each dimension (a unique "instance" of the underlying Field type). Each node in the triangle represents a vector sub-space (linear subspace).

Thus, every triangle (array of capacity n) represents a possible Vector Space, and the nodes in each slice triangle represent some of its possible linear subspaces.