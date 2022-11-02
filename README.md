# Experiments with `-rdynamic` and cargo

This works

```
> RUSTFLAGS="-C link-args=-rdynamic" cargo build
   Compiling rust-rdynamic v0.1.0 (/home/folkertdev/roc/rust-rdynamic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
> readelf --dyn-syms target/debug/rust-rdynamic | grep "look"
  1427: 00000000000683d0     6 FUNC    GLOBAL DEFAULT   14 look_at_me
```

but adding linker flags in `[build]` in the local `.cargo/config` does not get picked up. I believe this only works in the global `~/.cargo/config`. So this does nothing:

```
[build]
rustflags = [ "-C", "link-arg=-rdynamic" ]
```

Instead, we can use

```
[target.'cfg(all())']
rustflags = [ "-C", "link-arg=-rdynamic" ]
```

and then the symbol is available 

```
> cargo build
   Compiling rust-rdynamic v0.1.0 (/home/folkertdev/roc/rust-rdynamic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
> readelf --dyn-syms target/debug/rust-rdynamic | grep "look"
```

But this did not work on windows for me, I got:

``` 
note: x86_64-w64-mingw32-gcc: error: unrecognized command line option '-rdynamic'
```

Weird, because `gcc` should have that option. Anyway, some googling led me to

```
[target.'cfg(unix)']
rustflags = [ "-C", "link-arg=-rdynamic"]

[target.'cfg(windows)']
rustflags = [ "-C", "link-arg=-Wl,--export-all-symbols"]
```

Which seems to do the trick:

```
> objdump -x rust-rdynamic.exe | grep "look_at_me"
	[2315] look_at_me
[282](sec  1)(fl 0x00)(ty  20)(scl   2) (nx 0) 0x00000000000007f0 look_at_me
```
