#distributed

>[!Paradigm]
>Example about a program to count words:
>*How to count all the words (every single word) that occurs on a paragraph*
>The best approach maybe the hash map, but consider about the size of the map and the time to complete this task.
>It makes the foundation, the motivation about large-scale data processing.

## The problem with Large-Scale data processing

There are several ways to tackle this problem, these most 2 common ways are:
- Indexing algorithms: which web pages contain a certain word.
- Link Reversal: which pages links to this page.
- Sorting large data alphabetically

### Indexing algorithm

Take inputs as billions of documents, distributed across machines + link reversal and produce the output: incoming links for each document.
So the requirements are:
- Scalable
- Distributed
- Effective
- ...

## Map Reduce (or MapReduce)
>[!Define]

Programming model inspired by functional language primitives (LISP).
- Read a lot of data
- **Map** extracts data and just take the things you want
- **Reduce** aggregate and take meaningful information (include grouping)
- Produce output
Programmer specifies 2 functions:
- Map the raw input to key-value for the intermediate value.
- Reduce to combines all intermediate values under the same particular key.
- 