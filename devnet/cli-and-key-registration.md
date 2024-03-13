# Key Registration

The Gevulot devnet is permissioned. All users need to register a key for allowlisting to get access. You will find instructions for generating and registering a key below.\
\
**Prerequisites to be installed**

* Rust
* Cargo
* C Development Tools (GCC, Make, Binutils etc.)
* openssl-devel (libssl-dev in Debian derivatives)
* protobuf-compiler

#### **Step-by-step guide to generate your keys with Gevulot CLI**&#x20;

Follow the below commands to generate your secret keys:

1.  Install the Gevulot CLI:

    ```
    cargo install --git https://github.com/gevulotnetwork/gevulot.git gevulot-cli
    ```

    (Note: It should be in **$PATH**, and can be issued by calling **gevulot-cli**.)
2.  Generate a new cryptographic key pair. This will be used to sign transactions.

    ```
    gevulot-cli generate-key
    ```
3. A local file **localkey.pki** is created in the current directory and the public key is printed for copying.

#### **Key registration**

Once your key is generated, we require each participant to register their key with us. This is necessary for us to set up and manage the allowlist and to ensure security. **Once available, please submit your key through** [**the registration form here**](https://airtable.com/appS1ebiXFs8H4OP5/pagVuySwNkMe95tIi/form)**.**\
\
We will inform you once your key has been allowlisted.

\
