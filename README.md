# SvelteKit + Rust/Wasm Tutorial

For this tutorial, we will create a rust/wasm project called "wasm-lib" and a SvelteKit web application called "svelte-app" that uses "wasm-lib".

The "wasm-lib" project will be added to "svelte-app" as a [git submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules), allowing us to keep it as a separate git repository that can be used in multiple projects, without publishing the package.

#  1. Reading
- [Rust and WebAssembly Book](https://rustwasm.github.io/docs/book/)
- [SvelteKit Docs](https://kit.svelte.dev/docs/introduction)

# 2. Requirements
- [npm](https://docs.npmjs.com/getting-started) 
   comes with [nodejs](https://nodejs.org/en)
- [Rust Toolchain](https://www.rust-lang.org/tools/install)
- [wasm-pack](https://rustwasm.github.io/wasm-pack/installer/)
- [cargo-generate](https://github.com/cargo-generate/cargo-generate)

# 3. Create the Rust project

```console
cargo generate --git https://github.com/rustwasm/wasm-pack-template --name wasm-lib
```
## 3.1. 💡  Create a remote repository (e.g. GitHub)
Optional step (repo can be kept local)

[Adding locally hosted code to GitHub](https://docs.github.com/en/migrations/importing-source-code/using-the-command-line-to-import-source-code/adding-locally-hosted-code-to-github)

The template comes with git initialized, make a commit, and push the changes to the remote.

We will add this repository as a Git Submodule to the SvelteKit project later. 



## 3.2. Hello World

The template comes with bindings for alert() and greet().

Let's add binding and macro for console.log() because why not? Check [this](https://rustwasm.github.io/wasm-bindgen/examples/console-log.html) for more.

lib.rs
```rust
//wasm-lib\src\lib.rs

// ...

#[wasm_bindgen]
extern "C" {
    // ...
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
}

macro_rules! console_log {
    ($($t:tt)*) => (log(&format_args!($($t)*).to_string()))
}

#[wasm_bindgen]
pub fn greet() {
    // ...
    console_log!("Hello, console!");
}
```

## 3.3. Build

```console
wasm-pack build --target web
```
We don't need that yet, just a sanity check.

# 4. Create the SvelteKit Project
Check the [SvelteKit Docs](https://kit.svelte.dev/docs/introduction)

❗️ **Not in the rust (wasm-lib) project!** ❗️

```console
npm create svelte@latest svelte-app
```
- Skeleton Project

Optional:

- Yes, using Typescript syntax 
- Add ESLint ...
- Add Prettier ...
  
```console
cd svelte-app
npm install
git init
git add -A
git commit -m "initial commit"
```

Sanity Check:
```console
npm run dev -- --open
```
 ## 4.1. Add rust project as a submodule

Add the wasm-lib repo as a git submodule, using the remote repository created in [step 3.1](#31-💡-create-a-remote-repository-eg-github), or use the local path.

 ```console
git submodule add https://github.com/Leangeful/wasm-lib
```
This clones the rust repo into our SvelteKit project.

## 4.2. Install and Configure vite-plugin-wasm-pack

```console
npm install vite-plugin-wasm-pack -D
```

vite.config.ts
```typescript
//svelte-app\vite.config.ts

// ...
import wasmPack from 'vite-plugin-wasm-pack';

// ...
plugins: [sveltekit(), wasmPack('./wasm-lib')]
// ...
```

## 4.3. Build wasm-lib
Like we did in [step 3.3](#33-build), however this time in our svelte-app.

For convenience, we add the build script to our package.json.
```typescript
//svelte-app\package.json

// ...
"scripts": {
		"wasm": "wasm-pack build ./wasm-gfx --target web",
		
// ...
}
```

We run this script with:
```console
npm run wasm
```

## 4.4. Import wasm-lib