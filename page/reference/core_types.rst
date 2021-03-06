.. _doc_core_types:

核心类型
==========

Godot有一大组的类和模板来构成它的核心，并且所有东西都是建立在它们之上的。

这个参考将会尝试列举它们以便更好的理解它们。

定义
-----------

Godot用了标准C98的数据类型，比如“uint8_t”、“uint32_t”、“int64_t”等，现如今被所有编辑器所支持。为此重新发明一套体系可并不有趣，因为这回使得代码非常难读。

总的来说，除非使用大型的结构或者数组，否则不用在意为了指定的任务使用最有效率的数据类型。如果不必要的话“int”能贯穿整张代码使用。
In general, care is not taken to use the most efficient datatype for a
given task unless using large structures or arrays. ``int`` is used
through most of the code unless necessary. This is done because nowadays
every device has at least a 32 bits bus and can do such operations in
one cycle. It makes code more readable too.

For files or memory sizes, ``size_t`` is used, which is warranted to be
64 bits.

For Unicode characters, CharType instead of wchar_t is used, because
many architectures have 4 bytes long wchar_t, where 2 bytes might be
desired. However, by default, this has not been forced and CharType maps
directly to wchar_t.

References:
~~~~~~~~~~~

-  `core/typedefs.h <https://github.com/godotengine/godot/blob/master/core/typedefs.h>`__

Memory model
------------

PC is a wonderful architecture. Computers often have gigabytes of RAM,
terabytes of storage and gigahertz of CPU, and when an application needs
more resources the OS will just swap out the inactive ones. Other
architectures (like mobile or consoles) are in general more limited.

The most common memory model is the heap, where an application will
request a region of memory, and the underlying OS will try to fit it
somewhere and return it. This often works best and is very flexible,
but over time and with abuse, this can lead to segmentation.

Segmentation slowly creates holes that are too small for most common
allocations, so that memory is wasted. There is a lot of literature
about heap and segmentation, so this topic will not be developed
further here. Modern operating systems use paged memory, which helps
mitigate the problem of segmentation but doesn't solve it.

However, in many studies and tests, it is shown that given enough
memory, if the maximum allocation size is below a given threshold in
proportion to the maximum heap size and proportion of memory intended to
be unused, segmentation will not be a problem over time as it will
remain constant. In other words, just leave 10-20% of your memory free
and perform all small allocations and you are fine.

Godot ensures that all objects that can be allocated dynamically are
small (less than a few kb at most). But what happens if an allocation is
too large (like an image or mesh geometry or large array)? In this case
Godot has the option to use a dynamic memory pool. This memory needs to
be locked to be accessed, and if an allocation runs out of memory, the
pool will be rearranged and compacted on demand. Depending on the need
of the game, the programmer can configure the dynamic memory pool size.

Allocating memory
-----------------

Godot has many tools for tracking memory usage in a game, specially
during debug. Because of this, the regular C and C++ library calls
should not be used. Instead, a few other ones are provided.

For C-style allocation, Godot provides a few macros:

.. code:: cpp

    memalloc()
    memrealloc()
    memfree()

These are equivalent to the usual malloc, realloc, free of the standard
library.

For C++-style allocation, special macros are provided:

.. code:: cpp

    memnew( Class / Class(args) )
    memdelete( instance )

    memnew_arr( Class , amount )
    memdelete_arr( pointer to array )

which are equivalent to new, delete, new[] and delete[].

memnew/memdelete also use a little C++ magic and notify Objects right
after they are created, and right before they are deleted.

For dynamic memory, the DVector<> template is provided. Just use it
like:

.. code:: cpp

    DVector<int>

DVector is just a standard vector class, it can be accessed using the []
operator, but that's probably slow for large amount of accesses (as it
has to lock internally). A few helpers exist for this:

.. code:: cpp

    DVector<int>::Read r = dvector.read()
    int someint = r[4]

and

.. code:: cpp

    DVector<int>::Write w = dvector.write()
    w[4]=22;

respectively. These allow fast read/write from DVectors and keep it
locked until they go out of scope.

References:
~~~~~~~~~~~

-  `core/os/memory.h <https://github.com/godotengine/godot/blob/master/core/os/memory.h>`__
-  `core/dvector.h <https://github.com/godotengine/godot/blob/master/core/dvector.h>`__

Containers
----------

Godot provides also a set of common containers:

-  Vector
-  List
-  Set
-  Map

The are very simple and aim to be as minimal as possible, as templates
in C++ are often inlined and make the binary size much fatter, both in
debug symbols and code. List, Set and Map can be iterated using
pointers, like this:

.. code:: cpp

    for(List<int>::Element *E=somelist.front();E;E=E->next()) {
        print_line(E->get()); //print the element
    }

The Vector<> class also has a few nice features:

-  It does copy on write, so making copies of it is cheap as long as
   they are not modified.
-  It supports multi-threading, by using atomic operations on the
   reference counter.

References:
~~~~~~~~~~~

-  `core/vector.h <https://github.com/godotengine/godot/blob/master/core/vector.h>`__
-  `core/list.h <https://github.com/godotengine/godot/blob/master/core/list.h>`__
-  `core/set.h <https://github.com/godotengine/godot/blob/master/core/set.h>`__
-  `core/map.h <https://github.com/godotengine/godot/blob/master/core/map.h>`__

String
------

Godot also provides a String class. This class has a huge amount of
features, full Unicode support in all the functions (like case
operations) and utf8 parsing/extracting, as well as helpers for
conversion and visualization.

References:
~~~~~~~~~~~

-  `core/ustring.h <https://github.com/godotengine/godot/blob/master/core/ustring.h>`__

StringName
----------

StringNames are like a String, but they are unique. Creating a
StringName from a string results in a unique internal pointer for all
equal strings. StringNames are really useful for using strings as
identifier, as comparing them is basically comparing a pointer.

Creation of a StringName (specially a new one) is slow, but comparison
is fast.

References:
~~~~~~~~~~~

-  `core/string_db.h <https://github.com/godotengine/godot/blob/master/core/string_db.h>`__

Math types
----------

There are several linear math types available in the core/math
directory, they are basically just that.

References:
~~~~~~~~~~~

-  `core/math <https://github.com/godotengine/godot/blob/master/core/math>`__

NodePath
--------

This is a special datatype used for storing paths in a scene tree and
referencing them fast.

References:
~~~~~~~~~~~

-  `core/path_db.h <https://github.com/godotengine/godot/blob/master/core/path_db.h>`__

RID
---

RIDs are resource IDs. Servers use these to reference data stored in
them. RIDs are opaque, meaning that the data they reference can't be
accessed directly. RIDs are unique, even for different types of
referenced data.

References:
~~~~~~~~~~~

-  `core/rid.h <https://github.com/godotengine/godot/blob/master/core/rid.h>`__
