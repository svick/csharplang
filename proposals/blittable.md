# Blittable Types

* [x] Proposed
* [ ] Prototype
* [ ] Implementation
* [ ] Specification

## Summary
[summary]: #summary

The blittable feature will give language enforcement to the class of types known as "unmanaged types" in the C# language spec.  This is defined in section 18.2 as a type which is not a reference type and doesn't contain reference type fields at any level of nesting.  

## Motivation
[motivation]: #motivation

The primary motivation is to make it easier to author low level interop code in C#.  Blittable types are commonly used in interop code because they can be marshaled directly between native and managed memory. The bits can be moved as is 

This feature also allows developers to be more declarative about their interop types.  Today there is no way to assert and enforce a particular type is interop compatible (free of references).  Instead developers must resort to manual inspection on structures they consume.  

Even when completed properly there is no guarantee that those structures will remain interop compatible in future releases.  The owning developer may never have meant for the struct to be interop and hence wouldn't think it a break to add a reference type to the definition.  

**need** spooky action at a distance here 

The blittable feature helps avoid this by making the intent to be reference free declarative and enforcable.

This feature also allows developers to declaratively mark their types as suitable for interop.  Today there is no way to mark a type in this manner and developers need to resort to guessing based on the structure of the type involved.  

This feature allows developers to declaratively mark their types as useful in interop scenarios.  

Additionally having bilttable types in the language will open the door to a number of other features in the language:

- Fixed sized buffers: **research** why can't we do this today even with reference types?  Seems fine now that we have ref returns. 
- Span and cast method.

## Detailed design
[design]: #detailed-design

The language will introduce a new declaration modifier named `blittable` that can be applied to `struct` definitions.  

``` c#
blittable struct Point 
{
    public int X;
    public int Y;
}
```

The compiler will enforce that the fields of such a struct definition fit one of the following categories:

- `sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `char`, `float`, `double`, `decimal`, or `bool`.
- Any `enum` type
- Any pointer type which points to a `blittable` type
- Any user defined struct explicitly declared as `blittable`

Any type fitting the definition above, or any element in the list, is considered a valid `blittable` type.

``` c#
struct User
{
    string FirstName;
    string LastName;
}

blittable struct ItemData
{
    // Error: blittable struct cannot contain field of type User which is not blittable
    User User;
    int Id;
}
```

Note that a user defined struct must be explicitly declared as `blittable` in order to meet the requirements above.  This is required even if the struct otherwise meets the requirements of `blittable`. 

``` c#
struct SimplePoint
{
    public int X;
    public int Y;
}

blittable struct Data
{
    // Error: blittable struct cannot contain field of type SimplePoint which is not blittable
    SimplePoint Point;
}
```

The language will also support the ability to constrain generic type parameters to be `blittable` types.  

``` C#
void M<T>(T p) where T : blittable struct
{
    ...
}

M<Point>(); // Ok
M<User>();  // Error: Type User does not satisfy the blittable constraint.
```

One of the primary motivations for `blittable` structs is ease of interop.  As such it's imperative that such a type have it's field ordered in a sequential, or explicit, layout.  An auto layout of fields makes it impossible to reliably interop the data.  

The default today is for a sequential layout so this doesn't represent a substantial change.  However the compiler will make it illegal to mark such types as having an auto layout.  

``` c#
// Error: A blittable struct may not be marked with an auto layout
[StructLayout(LayoutKind.Auto)]
blittable struct LayoutExample 
{
    ...
}
```

## Drawbacks
[drawbacks]: #drawbacks

## Alternatives
[alternatives]: #alternatives

mention the F# unmanaged constraint 

## Unresolved questions
[unresolved]: #unresolved-questions

n/a

## Design meetings

n/a
