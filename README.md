# node-template-with-nicks-pallet-added
added nicks pallet to node template version 1.9 for answering a [SE question](https://substrate.stackexchange.com/questions/11468/substrate-tutorials-no-more-nicks-pallet).

1. Start with the substrate node template version 1.9.0, (im fixing the commit in the command):

    ```
    git clone git@github.com:substrate-developer-hub/substrate-node-template.git --branch main --single-branch --depth 1 --no-checkout && \
    cd substrate-node-template && \
    git checkout d70f8f9793cdd869571c21d7253faa50baaa5c5b
    ```

2. Make sure it compiles:

    ```
    cargo build --release
    ```

3. Copy nicks pallet [source code](https://github.com/paritytech/substrate/tree/master/frame/nicks) to /note-template/pallets/nicks/

4. In ./nicks/Cargo.toml, change the declarations that use local paths to git paths:

    ```toml
    #...
    [dependencies]
    codec = { package = "parity-scale-codec", version = "3.6.1", default-features = false, features = ["derive"] }
    scale-info = { version = "2.5.0", default-features = false, features = ["derive"] }
    frame-support = { git = "https://github.com/paritytech/polkadot-sdk.git", tag = "polkadot-v1.9.0", default-features = false }
    frame-system = { git = "https://github.com/paritytech/polkadot-sdk.git", tag = "polkadot-v1.9.0", default-features = false }
    sp-io = { git = "https://github.com/paritytech/polkadot-sdk.git", tag = "polkadot-v1.9.0", default-features = false }
    sp-runtime = { git = "https://github.com/paritytech/polkadot-sdk.git", tag = "polkadot-v1.9.0", default-features = false }
    sp-std = { git = "https://github.com/paritytech/polkadot-sdk.git", tag = "polkadot-v1.9.0", default-features = false }

    [dev-dependencies]
    pallet-balances = { git = "https://github.com/paritytech/polkadot-sdk.git", tag = "polkadot-v1.9.0" }
    sp-core = { git = "https://github.com/paritytech/polkadot-sdk.git", tag = "polkadot-v1.9.0" }
    #...
    ```

5. In ./runtime/Cargo.toml, add to dependencies:

    ```toml
    # The pallet in this template.
    pallet-template = { path = "../pallets/template", default-features = false }
    # Adding nicks pallet.
    pallet-nicks = { path = "../pallets/nicks", default-features = false }
    ```
    and "pallet-nicks/std" to the list of features; 

    ```toml
    std = [
	#...
	"pallet-template/std",
	"pallet-nicks/std",
	#...
    ]
    ```
    
6. In the project root ./Cargo.toml, add pallets/nicks to the list of members:

    ```toml
    [workspace]
    members = [
    "node",
    "pallets/template",
    "pallets/nicks",
    "runtime",
    ]
    ```

7. Check all ok until this stage:

    ```
    cargo check -p node-template-runtime --release
    ```

8. Implement the configuration for Nicks insisde the ./runtime/src/lib.rs

    * Add it to mod runtime:

    ```rust
    #[frame_support::runtime]
    mod runtime {
        ...

        // Include the custom logic from the pallet-template in the runtime.
        #[runtime::pallet_index(7)]
        pub type TemplateModule = pallet_template;

        // Add nicks pallet
        #[runtime::pallet_index(8)]
        pub type NicksModule = pallet_nicks;
    }
    ```

    * And also include the pallet config implementation:

    ```rust
    /// Configure the pallet-template in pallets/template.
    impl pallet_template::Config for Runtime {
        type RuntimeEvent = RuntimeEvent;
        type WeightInfo = pallet_template::weights::SubstrateWeight<Runtime>;
    }

    /// Configure the pallet-nicks
    impl pallet_nicks::Config for Runtime {
        // The Balances pallet implements the ReservableCurrency trait.
        // `Balances` is defined in `construct_runtime!` macro.
        type Currency = Balances;
        
        // Set ReservationFee to a value.
        type ReservationFee = ConstU128<100>;
        
        // No action is taken when deposits are forfeited.
        type Slashed = ();
        
        // Configure the FRAME System Root origin as the Nick pallet admin.
        // https://paritytech.github.io/substrate/master/frame_system/enum.RawOrigin.html#variant.Root
        type ForceOrigin = frame_system::EnsureRoot<AccountId>;
        
        // Set MinLength of nick name to a desired value.
        type MinLength = ConstU32<8>;
        
        // Set MaxLength of nick name to a desired value.
        type MaxLength = ConstU32<32>;
        
        // The ubiquitous event type.
        type RuntimeEvent = RuntimeEvent;
    }
    ```

8. Compile:

    ```
    cargo build --release
    ```

9. Run it in dev mode for testing:

    ```
    ./target/release/node-template --dev
    ```
    

