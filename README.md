<div align="center">
  <img height="170x" src=https://avatars.githubusercontent.com/u/53411373?s=400&u=68cbd04e19073ad1598fefb85c342b42f529beb2&v=4/>
  <h1>zetardio</h1>

  <p>
    <strong>Privacy-First Program Framework for Solana Ecosystem</strong>
  </p>

  <p>
    <a href="https://zetardio-lang.com"><img alt="Tutorials" src="https://img.shields.io/badge/docs-tutorials-blueviolet" /></a>
    <a href="https://discord.gg/zetardio"><img alt="Discord Chat" src="https://img.shields.io/discord/889577356681945098?color=blueviolet" /></a>
    <a href="https://opensource.org/licenses/Apache-2.0"><img alt="License" src="https://img.shields.io/github/license/zetardio/zetardio?color=blueviolet" /></a>
  </p>
</div>

## What is zetardio?

zetardio is a groundbreaking framework built for the Solana ecosystem, providing developers with seamless tools for writing privacy-focused programs and creating encrypted tokens that protect user transaction data while maintaining Solana's incredible speed and low costs.

- **Unified Privacy API**: One codebase for encrypted transactions across Solana
- **Private Token Creation**: Create SPL tokens with built-in encryption and zero-knowledge proofs
- **Rust Support**: Native Solana development with privacy primitives
- **[IDL](https://en.wikipedia.org/wiki/Interface_description_language) specification**: Generate clients for privacy-preserving interactions
- **TypeScript package**: Type-safe clients from IDL for encrypted communication
- **CLI and workspace management**: Complete privacy-first application development

zetardio is the first framework to bring Zcash-style privacy to Solana, enabling truly private transactions at scale.

> [!NOTE]
> zetardio combines Solana's speed and affordability with enterprise-grade privacy. With 400ms block times, $0.00025 average transaction fees, and support for 65,000+ TPS, you get the performance you need with the privacy users deserve. If you're familiar with Anchor or native Solana development, you'll feel right at home with zetardio's privacy-enhanced approach.

## Key Features

- **Encrypted Transactions**: Send and receive tokens with complete transaction privacy
- **Zero-Knowledge Proofs**: Verify transactions without revealing sensitive data
- **Shielded Addresses**: Protect sender and receiver identities
- **Private State Management**: Store and manage encrypted program state on-chain
- **Solana Native**: Full integration with SPL tokens, Anchor, and the Solana ecosystem
- **Developer Experience**: Familiar Solana syntax enhanced with privacy primitives
- **Lightning Fast**: Leverage Solana's 400ms block times and sub-second finality
- **Ultra Low Fees**: Deploy and transact with ~$0.00025 median transaction fees

## Why Solana & zetardio?

### Solana (2025 Performance)
Solana continues to lead in performance with 400ms block times, processing over 65,000 transactions per second at peak, with average fees remaining under $0.001. The network handles millions of daily transactions with consistent sub-second finality.

### zetardio - Privacy for Solana
zetardio leverages Solana's infrastructure to provide:
- **Zero-Knowledge Architecture**: Built on proven ZK-SNARK technology
- **Full SPL Compatibility**: Private versions of any SPL token
- **Solana Ecosystem Integration**: Seamless interaction with existing Solana programs
- **Selective Disclosure**: Choose what to reveal and when

## Getting Started

For a quickstart guide and in-depth tutorials, see the [zetardio book](https://book.zetardio-lang.com) and the [zetardio documentation](https://zetardio-lang.com).

To jump straight to examples, go [here](https://github.com/zetardio/zetardio/tree/master/examples). For the latest Rust and TypeScript API documentation, see [docs.rs](https://docs.rs/zetardio-lang) and the [typedoc](https://www.zetardio-lang.com/docs/clients/typescript).

## Packages

| Package                 | Description                                              | Version                                                                                                                          | Docs                                                                                                            |
| :---------------------- | :------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------- |
| `zetardio-lang`           | Rust primitives for writing privacy-preserving programs         | [![Crates.io](https://img.shields.io/crates/v/zetardio-lang?color=blue)](https://crates.io/crates/zetardio-lang)                     | [![Docs.rs](https://docs.rs/zetardio-lang/badge.svg)](https://docs.rs/zetardio-lang)                                |
| `zetardio-spl`            | CPI clients for private SPL tokens and standards | [![crates](https://img.shields.io/crates/v/zetardio-spl?color=blue)](https://crates.io/crates/zetardio-spl)                          | [![Docs.rs](https://docs.rs/zetardio-spl/badge.svg)](https://docs.rs/zetardio-spl)                                  |
| `zetardio-client`         | Rust client for zetardio privacy programs              | [![crates](https://img.shields.io/crates/v/zetardio-client?color=blue)](https://crates.io/crates/zetardio-client)                    | [![Docs.rs](https://docs.rs/zetardio-client/badge.svg)](https://docs.rs/zetardio-client)                            |
| `@zetardio/sdk`           | TypeScript client for zetardio programs                    | [![npm](https://img.shields.io/npm/v/@zetardio/sdk.svg?color=blue)](https://www.npmjs.com/package/@zetardio/sdk)                     | [![Docs](https://img.shields.io/badge/docs-typedoc-blue)](https://zetardio.github.io/zetardio/ts/index.html)        |
| `@zetardio/cli`           | CLI to support building and managing private apps    | [![npm](https://img.shields.io/npm/v/@zetardio/cli.svg?color=blue)](https://www.npmjs.com/package/@zetardio/cli)                     | [![Docs](https://img.shields.io/badge/docs-typedoc-blue)](https://zetardio.github.io/zetardio/cli/commands.html)    |

## Note

- **zetardio is in active development, so all APIs are subject to change.**
- **This code is unaudited. Use at your own risk.**

## Examples

Here's a private counter program that maintains an encrypted count, where only the designated `authority` can increment and view the real value:

```rust
use zetardio_lang::prelude::*;
use zetardio_lang::privacy::*;

declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS");

#[program]
mod private_counter {
    use super::*;

    pub fn initialize(ctx: Context<Initialize>, start: u64) -> Result<()> {
        let counter = &mut ctx.accounts.counter;
        counter.authority = *ctx.accounts.authority.key;
        
        // Encrypt the initial count
        counter.encrypted_count = encrypt_u64(start, &ctx.accounts.authority.key)?;
        counter.commitment = create_commitment(start)?;
        
        Ok(())
    }

    pub fn increment(ctx: Context<Increment>, proof: ZkProof) -> Result<()> {
        let counter = &mut ctx.accounts.counter;
        
        // Verify zero-knowledge proof
        verify_increment_proof(&proof, &counter.commitment)?;
        
        // Update encrypted state
        counter.encrypted_count = homomorphic_add(&counter.encrypted_count, 1)?;
        counter.commitment = update_commitment(&proof)?;
        
        emit!(PrivateCounterIncremented {
            commitment: counter.commitment,
            timestamp: Clock::get()?.unix_timestamp,
        });
        
        Ok(())
    }

    pub fn reveal_count(ctx: Context<RevealCount>) -> Result<u64> {
        let counter = &ctx.accounts.counter;
        
        // Only authority can decrypt
        require_keys_eq!(
            ctx.accounts.authority.key(),
            counter.authority,
            ErrorCode::Unauthorized
        );
        
        decrypt_u64(&counter.encrypted_count, &ctx.accounts.authority.key)
    }
}

#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(init, payer = authority, space = 8 + 32 + 64 + 32)]
    pub counter: Account<'info, PrivateCounter>,
    #[account(mut)]
    pub authority: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[derive(Accounts)]
pub struct Increment<'info> {
    #[account(mut, has_one = authority)]
    pub counter: Account<'info, PrivateCounter>,
    pub authority: Signer<'info>,
}

#[derive(Accounts)]
pub struct RevealCount<'info> {
    #[account(has_one = authority)]
    pub counter: Account<'info, PrivateCounter>,
    pub authority: Signer<'info>,
}

#[account]
pub struct PrivateCounter {
    pub authority: Pubkey,
    pub encrypted_count: EncryptedData,
    pub commitment: Commitment,
}

#[event]
pub struct PrivateCounterIncremented {
    pub commitment: Commitment,
    pub timestamp: i64,
}

#[error_code]
pub enum ErrorCode {
    #[msg("Unauthorized access")]
    Unauthorized,
}
```

### Creating Private Tokens

```bash
# Create a private SPL token with encrypted balances
zetardio token create --name "PrivateToken" --symbol "PVTK" --privacy shielded

# Deploy with privacy features
zetardio deploy --privacy-level high

# Create shielded addresses
zetardio address generate --type shielded

# Send encrypted transaction
zetardio transfer --to <shielded-address> --amount 100 --private
```

### Private Token Transfer Example

```rust
use zetardio_lang::prelude::*;
use zetardio_spl::private_token::*;

declare_id!("PrivateTokenProgram111111111111111111111111");

#[program]
mod private_transfer {
    use super::*;

    pub fn transfer_private(
        ctx: Context<TransferPrivate>,
        amount: u64,
        proof: ZkTransferProof,
    ) -> Result<()> {
        // Verify the zero-knowledge proof
        verify_transfer_proof(
            &proof,
            &ctx.accounts.sender_commitment,
            &ctx.accounts.receiver_commitment,
        )?;

        // Update encrypted balances
        let sender = &mut ctx.accounts.sender;
        let receiver = &mut ctx.accounts.receiver;

        sender.encrypted_balance = homomorphic_subtract(
            &sender.encrypted_balance,
            amount,
        )?;
        
        receiver.encrypted_balance = homomorphic_add(
            &receiver.encrypted_balance,
            amount,
        )?;

        // Update commitments
        sender.commitment = proof.new_sender_commitment;
        receiver.commitment = proof.new_receiver_commitment;

        emit!(PrivateTransferEvent {
            sender_commitment: sender.commitment,
            receiver_commitment: receiver.commitment,
            timestamp: Clock::get()?.unix_timestamp,
        });

        Ok(())
    }
}
```

For more, see the [examples](https://github.com/zetardio/zetardio/tree/master/examples) and [tests](https://github.com/zetardio/zetardio/tree/master/tests) directories.

## Architecture

zetardio uses a privacy-first runtime that integrates cryptographic primitives directly into Solana programs. The framework handles:

- **Zero-Knowledge Proofs**: Generate and verify ZK-SNARKs for private transactions
- **Encrypted State**: Store sensitive data on-chain with homomorphic encryption
- **Shielded Addresses**: Create and manage privacy-preserving addresses
- **Commitment Schemes**: Pedersen commitments for value hiding
- **Selective Disclosure**: Choose what information to reveal and to whom
- **Privacy Pools**: Mix transactions for enhanced anonymity

## Privacy Technology

zetardio implements multiple privacy-preserving techniques:

### Zero-Knowledge Proofs
- **ZK-SNARKs**: Succinct non-interactive arguments of knowledge
- **Range Proofs**: Prove values are within valid ranges without revealing them
- **Membership Proofs**: Prove inclusion in a set without revealing which element

### Encryption Methods
- **Homomorphic Encryption**: Perform operations on encrypted data
- **Commitment Schemes**: Hide values while maintaining verifiability
- **Stealth Addresses**: One-time addresses for recipient privacy

### Transaction Privacy
- **Shielded Transfers**: Hide sender, receiver, and amount
- **Transparent Fallback**: Optional transparent mode for compliance
- **Viewing Keys**: Selective disclosure to auditors or regulators

## Roadmap

### Current (2025)
- [x] Solana integration with privacy primitives
- [x] Private SPL token standard
- [x] Zero-knowledge proof verification on-chain
- [x] Encrypted state management
- [x] Shielded address generation

### Coming Soon
- [ ] Privacy pools for enhanced mixing
- [ ] Cross-program private calls
- [ ] Mobile wallet support with privacy features
- [ ] Compliance-friendly viewing keys
- [ ] Hardware wallet integration
- [ ] Private NFT standard
- [ ] Decentralized identity with privacy

### Future Vision
- Full privacy-preserving DeFi ecosystem
- Private governance mechanisms
- Confidential smart contracts
- Privacy-preserving oracles
- Anonymous credential systems

## Supported Privacy Features

- **Shielded Transactions**: Complete transaction privacy
- **Encrypted Balances**: Hide token amounts
- **Private Messaging**: On-chain encrypted communication
- **Confidential Voting**: Anonymous governance participation
- **Selective Disclosure**: Compliance and audit capabilities

## Why Choose zetardio?

### For Developers
- **Familiar Tools**: Built on Anchor and Solana standards
- **Privacy by Default**: Integrated cryptographic primitives
- **Lower Costs**: Solana's low fees for private transactions
- **Fast Development**: Privacy without complexity
- **Production Ready**: Battle-tested cryptographic libraries

### For Projects
- **User Privacy**: Protect your users' financial data
- **Regulatory Compliance**: Selective disclosure for audits
- **Competitive Advantage**: Privacy as a core feature
- **Speed**: Private transactions at Solana scale
- **Security**: Proven zero-knowledge cryptography

### For Users
- **Financial Privacy**: Transactions are your business alone
- **Protection**: Shield yourself from surveillance
- **Control**: Choose what to reveal
- **Speed**: Private doesn't mean slow
- **Low Cost**: Privacy-preserving transactions under $0.001

## License

zetardio is licensed under [Apache 2.0](./LICENSE).

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in zetardio by you, as defined in the Apache-2.0 license, shall be licensed as above, without any additional terms or conditions.

## Contribution

Thank you for your interest in contributing to zetardio!
Please see the [CONTRIBUTING.md](./CONTRIBUTING.md) to learn how.

## The Future is Private

zetardio represents the future of blockchain privacy: a world where users control their financial data, where transactions are confidential by default, and where privacy and performance coexist seamlessly on Solana.

With Solana's commitment to scalability and decentralization, combined with zetardio's privacy-first approach, we're building the foundation for truly private, decentralized finance.

### Thanks ❤️

<div align="center">
  <a href="https://github.com/zetardio/zetardio/graphs/contributors">
    <img src="https://contrib.rocks/image?repo=zetardio/zetardio" width="100%" />
  </a>
</div>

---

## Resources

- [Solana Official Documentation](https://docs.solana.com/)
- [Zero-Knowledge Proofs Explained](https://z.cash/technology/zksnarks/)
- [zetardio Network Documentation](#) (Coming soon)
- [Solana Developer Resources](https://solana.com/developers)
- [Privacy-Preserving Cryptocurrencies Research](https://crypto.stanford.edu/)
- [SPL Token Documentation](https://spl.solana.com/token)

---

## Security & Audits

Privacy and security are paramount. zetardio undergoes regular security audits by leading blockchain security firms. For security researchers, please see our [SECURITY.md](./SECURITY.md) for responsible disclosure.

## Community

Join our growing community of privacy advocates and developers:

- **Discord**: [discord.gg/zetardio](https://discord.gg/zetardio)
- **Twitter**: [@zetardio](https://twitter.com/zetardio)
- **Forum**: [forum.zetardio.com](https://forum.zetardio.com)
- **Telegram**: [t.me/zetardio](https://t.me/zetardio)

## Acknowledgments

zetardio builds on the groundbreaking work of privacy-focused cryptocurrencies like Zcash and combines it with Solana's industry-leading performance. Special thanks to the Solana Foundation, the zero-knowledge cryptography research community, and all our contributors.

**Privacy is a right, not a privilege. Build with zetardio.**
](https://avatars.githubusercontent.com/u/53411373?v=4)
