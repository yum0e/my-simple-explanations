# Explanation of the smart contract Adder

How to store an on-chain variable on the Elrond blockchain using a smart contract? This is what I am going to explain to you in this quick markdown which will focus on the Rust code of a very simple smart contract written by the Elrond team on this [github](https://github.com/ElrondNetwork/elrond-wasm-rs/blob/master/contracts/examples/adder/src/adder.rs), the Adder smart contract.

### But what is on-chain storage?

On-chain storage is data storage directly on the blockchain while off-chain storage is storage on an external server. As you will have understood, on-chain storage has a cost because you then force the various network validators to store your value, image, file...
In the case of NFTs for example, it is often preferable to store your image off-chain on a decentralized storage system, such as Arweave or IPFS for example, and to store only the url of your image in a on chain.

Pretty cool right ?
Let's dive in!

# The code

Here is the Rust code of the smart contract that interests us:

<blockquote>I assume that you have knowledge of Rust to follow this tutorial. If not, here is one of the best resource -> [Rust Book](https://doc.rust-lang.org/book/)<blockquote/>

```rust
#![no_std]

elrond_wasm::imports!();

/// One of the simplest smart contracts possible,
/// it holds a single variable in storage, which anyone can increment.
#[elrond_wasm::derive::contract]
pub trait Adder {
    #[view(getSum)]
    #[storage_mapper("sum")]
    fn sum(&self) -> SingleValueMapper<BigUint>;

    #[init]
    fn init(&self, initial_value: BigUint) {
        self.sum().set(initial_value);
    }

    /// Add desired amount to the storage variable.
    #[endpoint]
    fn add(&self, value: BigUint) {
        self.sum().update(|sum| *sum += value);
    }
}
```

# Setting up the environment

Rust's standard library provides many useful features, but supports various features of its host system: threads, networking, allocation on the heap... However, some systems do not have these features, and this is the case with the Elrond blockchain! But no worries, Rust can also work with these systems! To do this, we tell Rust that we don't want to use the standard library via an attribute: `#![no_std]`.

We will therefore make sure to put `#![no_std]` at the beginning of our different Rust codes when we code our smart contracts. As for `elrond_wasm::imports!();`, we use the imports macro which is simply used to import everything that will be useful to us for the creation of smart contracts in Rust on the Elrond blockchain.

# Smart contract creation

```rust
#[elrond_wasm::contract] // annotation needed for a smart contract
pub trait Adder {

    #[init] // annotation needed for the initialization function of our smart contract
    fn init(&self, initial_value: BigInt) {
        self.sum().set(&initial_value); // we store the value of initial_value
    }
}
```

The `#[elrond_wasm::contract]` annotation is used to indicate that the Adder [trait](https://doc.rust-lang.org/book/ch10-02-traits.html?highlight=trait) is the trait that contains all the logic of our smart contract. It is absolutely necessary to inform it to create our smart contract, it makes all the logic of the latter legitimate.

The `#[init]` annotation is used to indicate that the function following this annotation (in our case the init function) should only be called when our smart contract is deployed. Here, we see that our `init` function takes `&self` and `initial_value` as arguments and that it stores the value of `initial_value`. This storage is possible thanks to the call of our `sum` function.

# Store a variable

To understand how we store the desired value in line 13 of our program, let's look at the `sum` [storage mapper](https://docs.elrond.com/developers/best-practices/storage-mappers/).

```rust
    // useful function for storing and consulting our value
    #[view(getSum)]
    #[storage_mapper("sum")]
    fn sum(&self) -> SingleValueMapper<BigInt>;
```

Let's look at the `#[storage_mapper("sum")]` annotation.
This annotation tells us that our function is actually a StorageMapper called [SingleValueMapper](https://docs.rs/elrond-wasm/latest/elrond_wasm/storage/mappers/struct.SingleValueMapper.html) as can be seen in what our function returns. This SingleValueMapper allows, in short, to store a variable with `set` and to know the value stored with `get`. So when we call our `sum` function in `init`:

```rust
self.sum().set(&initial_value);
```

We indicate that we want to access our SingleValueMapper with `self.sum()` and then we call `set` with the argument `&initial_value`. We have therefore just stored the value `&initial_value` on the blockchain.
In the same way, to know the stored value we only have to write:

```rust
self.sum().get();
```

For more details, and to see the different possible StorageMappers, here is the [associated documentation](https://docs.elrond.com/developers/best-practices/storage-mappers/).
The second annotation `#[view(getSum)]` makes the function visible to smart contracts or external programs (just like a `view` function in Solidity).

# Update our variable

Now that you know how to store a variable and know its value once stored, all you have to do is update it. You can simply choose to update it by entering the value you want in `set`, but it seems more interesting to also be able to update it based on its already stored value.
To do this, you can use `update` which will take a closure as an argument.

```rust
    // Add the desired quantity to the storage variable
    #[endpoint] // same as #[view] annotation but here we make changes, we are not just 'viewing' the variable
    fn add(&self, value: BigInt) -> SCResult<()> {
        self.sum().update(|sum| *sum += value);

        Ok(())
    }
```

The `add` function meets our need with the use of `update` and the closure which takes `sum` as an argument (`sum` represents the value that we already have in storage) and returns `sum` plus a `value` that we provide as an argument when calling the `add` function.
So we have this behavior:

```rust
fn abc(&self) -> BigInt {
    let my_big_int = BigInt::from_i32(2)
    self.sum().set(&my_big_int); // we store 2 (a BigInt) which is the value of my_big_int
    self.add(BigInt::from_i32(4)); // we add 4 (a BigInt) with the add function
    self.sum().get() // we return 6 (a BigInt)
}
```

Finally, it's important to note that `add` returns a `SCResult<()>`. Here, this `SCResult<()>` is useful to know if an error occurs during the call to `add`. For more info on `SCResult<()>`, go to this [documentation](https://docs.rs/elrond-wasm/latest/elrond_wasm/types/enum.SCResult.html).

# Conclusion

You now know how to store an on-chain value and you know how to use the simplest StorageMapper from the Elrond documentation. You also know how to create a smart contract on the Elrond blockchain with a simple implementation. Feel free to give me feedback and follow me on twitter: [@yum0ee](https://twitter.com/yum0ee)
