# HashMap

To understand why a prime number was chosen, you have to know how hash-based collections such as `HashSet` and `HashMap` work.

Hash-based collections store elements in so-called buckets. A hash-based collection has a fixed number of buckets. When you put an object in the hash-based collection, then it uses the hash code of the object to determine in which bucket to put the object. In fact, I guess it uses the formula `hashCode() % numberOfBuckets` to compute which bucket to put the object in. (`%` is the modulo-operator).

When you want to find an object in a hash-based collection, this is what happens:

- Compute the hash code of the object to find.
- This tells the collection in which bucket to search.
- Look at all objects in the bucket and compare each of them with the object we're looking for, by calling `equals()` on the method in the bucket and the object we're looking for.

This algorithm is very efficient for finding an object in a collection - using the hash code, it knows immediately in which bucket to look, and it only needs to compare the object you're looking for with a few other objects, those in that particular bucket. Even if the collection contains thousands of objects, in general only a few comparisons are necessary to find a particular object.

There is one weak point: it only works well if elements in the collection are well-distributed among the buckets. Suppose you made a class with a hashCode() method that returns a fixed value. That's a correct implementation of hashCode(), but it defeats the efficiency of hash-based collections - all objects would be stored in the same bucket (because their hash codes are all the same), and finding an object in the collection would then mean that it would have to be compared to all objects in that single bucket.

So, to make hash-based collections work optimally, the objects you put in the collection should have a hashCode() method that causes the objects to be distributed among the buckets as evenly as possible.

HashSet and HashMap in Java have a number of buckets that is normally a power of 2 (by default, a HashMap has 16 buckets, for example).

By making the hash code of objects a prime number (or a multiple of a prime number) the objects will be distributed better over the buckets, because a prime number is not a multiple of the number of buckets (which is a power of 2), so hashCode() % numberOfBuckets won't quickly cycle back to the same number (which means the same bucket).
