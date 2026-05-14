
>[! Definition]
>We know that hash table is a look up table use to trace backward plain text from a hashed string. But what about rainbow table?

But let make a clarify: both hash table and rainbow table are used for finding the plain text of the hashed password. And yes, of course, they are also quite less effective to salted password. Because to finding the initial value, each table for each salt bit should be generated and it leads to unimaginable size of tables.

# 1. How's it look like

In the hash table, the format `P:H` is used, that is, for each password, a hashed value was stored in corresponding with it. But it will grow to terrabyte for only a simple string.

An entry in the rainbow table will be look like this:
```text
P1 --hash-> H1 --reduce-> P2 --hash-> H2
```
And only the first and the last are stored in the table. So that we can trace the in-between values by computation, it is a trade-off between space (storing capacity) and time (lookup time).

With that's principle, we know that with some *reduce* function, a hashed value can become a plain text, then continue to hash it. When approaching any hashed value, we can use that reduce function to recreate and match the full chain to know the password (take at most `k` steps, not to mention that each step requires an amount of computing). But surely, less space consuming than traditional brute-force table.

# 2. Lookup

Example scenario: got a hashed value `FFE0FA...`.

The original chain will be something like this: `password123 - FFE0FA - kitkat234 - B9A1D3`.

The value is not a last endpoint, so that we must check early position on the hashed chain. The reduce function will be applied. And note that for any same hashes, the reduce function will produce same plain text strings `R(FFE0FA...) -> kitkat234`.

So that when the produced string was hashed, we see that it was an endpoint, hence reconstructed the whole chain, stated that the password was "password123".

If the hash hadn’t matched at H1, we could have applied the reduction function again and repeated the process to check the next chain position. However, with a chain length of 2 hashes, there are only two positions to check (H1 and H2). If neither position matches, the hash isn’t covered by our rainbow table.

In reality, chains will be significantly longer, meaning that a lookup may take many steps of reduction and hashing before ending up with a plaintext.

Note also that production-grade rainbow tables such as those from [crackmyha.sh](https://crackmyha.sh) often use **dynamic reduction functions** that change based on the chain index. This helps minimize **chain merging**, which occurs when two chains produce the same hash at the same position.

## Reference

[Rainbow table explained](https://medium.com/@TimTrademark/rainbow-tables-and-hash-tables-are-not-the-same-5d7293cdacc9)
