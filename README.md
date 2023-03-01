# Pallet loose coupling example.

In this example `PalletTemplate` is loosely coupled to `StoragePallet` by `Storage` trait. To be more precise `PalletTemplate::store_something(origin: OriginFor<T>, something: u32)` calls `Storage::store(something: u32)` which is implemented by `PalletStorage` and stores `something` in `StoragePallet` storage.



`pallets/storage/src/lib.rs`

```rust
...
#[pallet::storage]
#[pallet::getter(fn something)]
pub type Something<T> = StorageValue<_, u32>;
...
pub trait Storage {
	fn store(something: u32);
}
...
impl<T: Config> Storage for Pallet<T> {
  fn store(something: u32) {
    <Something<T>>::put(something);
  }
}
...
```

`pallets/template/src/lib.rs`
```rust
...
#[pallet::call_index(0)]
#[pallet::weight(10_000 + T::DbWeight::get().writes(1).ref_time())]
pub fn store_something(origin: OriginFor<T>, something: u32) -> DispatchResult {
  let who = ensure_signed(origin)?;

  T::Storage::store(something);

  Self::deposit_event(Event::SomethingStored { something, who });
  Ok(())
}
...
```
`runtime/src/lib.rs`
```rust
...
impl pallet_template::Config for Runtime {
	type RuntimeEvent = RuntimeEvent;
	type Storage = pallet_storage::Pallet<Self>;
}

impl pallet_storage::Config for Runtime {
	type RuntimeEvent = RuntimeEvent;
}
...
construct_runtime!(
	pub struct Runtime
	where
		Block = Block,
		NodeBlock = opaque::Block,
		UncheckedExtrinsic = UncheckedExtrinsic,
	{
...
		TemplateModule: pallet_template,
		StorageModule: pallet_storage
	}
);
...


```









# Build
`cargo run -- --dev`
