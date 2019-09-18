# Programming Model

A client _app_ interacts with a Solana cluster by sending it _transactions_ with one or more _instructions_. The Solana _runtime_ passes those instructions to user-contributed _programs_. An instruction might, for example, tell a program to transfer _lamports_ from one _account_ to another or create an interactive contract that governs how lamports are transfered. Instructions are executed atomically. If any instruction is invalid, any changes made within the transaction are discarded.

## Deploying Programs to a Cluster

![SDK tools](https://github.com/solana-labs/solana/tree/c0ec2ca27a81d5429d32f27148cbbefc4edab999/book/src/img/sdk-tools.svg)

As shown in the diagram above a client creates a program and compiles it to an ELF shared object containing BPF bytecode and sends it to the Solana cluster. The cluster stores the program locally and makes it available to clients via a _program ID_. The program ID is a _public key_ generated by the client and is used to reference the program in subsequent transactions.

A program may be written in any programming language that can target the Berkley Packet Filter \(BPF\) safe execution environment. The Solana SDK offers the best support for C programs, which is compiled to BPF using the [LLVM compiler infrastructure](https://llvm.org).

## Storing State between Transactions

If the program needs to store state between transactions, it does so using _accounts_. Accounts are similar to files in operating systems such as Linux. Like a file, an account may hold arbitrary data and that data persists beyond the lifetime of a program. Also like a file, an account includes metadata that tells the runtime who is allowed to access the data and how. Unlike a file, the account includes metadata for the lifetime of the file. That lifetime is expressed in "tokens", which is a number of fractional native tokens, called _lamports_. Accounts are held in validator memory and pay "rent" to stay there. Each fullnode periodically scan all accounts and collects rent. Any account that drops to zero lamports is purged.

If an account is marked "executable", it will only be used by a _loader_ to run programs. For example, a BPF-compiled program is marked executable and loaded by the BPF loader. No program is allowed to modify the contents of an executable account.

An account also includes "owner" metadata. The owner is a program ID. The runtime grants the program write access to the account if its ID matches the owner. If an account is not owned by a program, the program is permitted to read its data and credit the account.

In the same way that a Linux user uses a path to look up a file, a Solana client uses public keys to look up accounts. To create an account, the client generates a _keypair_ and registers its public key using the CreateAccount instruction. Once registered, transactions reference account keys to grant programs access to accounts. The runtime grants programs read access by default. To grant write access, the client must either assign the account to a program or sign the transaction using the keypair's _secret key_. Since only the holder of the secret key can produce valid signatures matching the account's public key, the runtime recognizes the signature as authorization to modify account data or debit the account.

After the runtime executes each of the transaction's instructions, it uses the account metadata and transaction signatures to verify that none of the access rules were violated. If a program violates an access rule, the runtime discards all account changes made by all instructions and marks the transaction as failed.
