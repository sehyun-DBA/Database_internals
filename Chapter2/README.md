## Chapter 2. B-Tree Basics

In the previous chapter, we separated storage structures in two groups:
mutable and immutable ones, and identified immutability as one of the
core concepts influencing their design and implementation. Most of the
mutable storage structures use an in-place update mechanism. During
insert, delete, or update operations, data records are updated directly in
their locations in the target file.

Storage engines often allow multiple versions of the same data record to be
present in the database; for example, when using multiversion concurrency
control (see “Multiversion Concurrency Control”) or slotted page
organization (see “Slotted Pages”). For the sake of simplicity, for now we
assume that each key is associated only with one data record, which has a
unique location.

One of the most popular storage structures is a B-Tree. Many open source
database systems are B-Tree based, and over the years they’ve proven to
cover the majority of use cases.

B-Trees are not a recent invention: they were introduced by Rudolph
Bayer and Edward M. McCreight back in 1971 and gained popularity over
the years. By 1979, there were already quite a few variants of B-Trees.
Douglas Comer collected and systematized some of them [COMER79].
Before we dive into B-Trees, let’s first talk about why we should consider
alternatives to traditional search trees, such as, for example, binary search
trees, 2-3-Trees, and AVL Trees [KNUTH98]. For that, let’s recall what
binary search trees are.
