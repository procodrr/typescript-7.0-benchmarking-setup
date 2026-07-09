# TypeScript 7 vs TypeScript 6 Benchmark Setup on VS Code Repo

**The Ultimate TypeScript Course:**
[https://app.procodrr.com/web/checkout/69aac6e22e200ab5f21a0793](https://app.procodrr.com/web/checkout/69aac6e22e200ab5f21a0793)

**Coupon Code:** `TSGO`

---

# Goal

Set up the VS Code repository locally, install all required dependencies, run the local development version of VS Code, and benchmark:

1. TypeScript 7 native compiler using `tsc`
2. TypeScript 6 compiler using `tsc6`
3. CLI type-checking performance
4. Editor performance, including autocomplete, hover, go-to-definition, and find-all-references

---

# Official setup guide

The VS Code setup guide is not mainly in `CONTRIBUTING.md`. It is in the VS Code GitHub Wiki.

Official setup guide:
[https://github.com/microsoft/vscode/wiki/How-to-Contribute](https://github.com/microsoft/vscode/wiki/How-to-Contribute)

Raw markdown version:
[https://raw.githubusercontent.com/wiki/microsoft/vscode/How-to-Contribute.md](https://raw.githubusercontent.com/wiki/microsoft/vscode/How-to-Contribute.md)

The guide says to install Git, the correct Node.js version, Python for `node-gyp`, and a platform-specific C/C++ compiler toolchain. On Windows, it specifically requires Visual Studio 2022 Build Tools with C++ components and Spectre-mitigated libraries. ([GitHub][1])

---

# 1. Clone the VS Code repository

Use a path without spaces. The official guide warns that spaces in the path can cause issues when compiling native modules. ([GitHub][1])

```powershell
cd C:\Users\anura\learning-locally
git clone https://github.com/microsoft/vscode.git
cd vscode
```

Your repo path became:

```text
C:\Users\anura\learning-locally\vscode
```

---

# 2. Use the correct Node.js version

The VS Code repo uses `.nvmrc` to specify the correct Node version. In our setup, `.nvmrc` said:

```text
24.17.0
```

The official guide says to check `.nvmrc` before installing dependencies. ([GitHub][1]) The current `.nvmrc` value is `24.17.0`. ([GitHub][2])

Install and use it:

```powershell
nvm install 24.17.0
nvm use 24.17.0
node -v
npm -v
```

Expected:

```text
v24.17.0
```

Do not continue if `node -v` shows Node 25 or any other version.

---

# 3. Install Visual Studio 2022 Build Tools

On Windows, the VS Code repo needs native modules to be compiled through `node-gyp`.

Run this in Admin PowerShell:

```powershell
winget install --id Microsoft.VisualStudio.2022.BuildTools -e --source winget --override "--add Microsoft.VisualStudio.Component.Windows11SDK.22621 --add Microsoft.VisualStudio.Workload.VCTools --add Microsoft.VisualStudio.Component.VC.Runtimes.x86.x64.Spectre --add Microsoft.VisualStudio.Component.VC.ATL.Spectre --add Microsoft.VisualStudio.Component.VC.ATLMFC.Spectre"
```

In the Visual Studio Installer, make sure these are selected:

```text
Desktop development with C++
MSVC v143 C++ build tools
Windows 11 SDK
MSVC Spectre-mitigated libraries
C++ ATL with Spectre Mitigations
C++ MFC with Spectre Mitigations
```

You do not need to click **Launch** after Build Tools installation. Just close the installer and continue in PowerShell.

---

# 4. Python requirement

Python is required because native modules are built through `node-gyp`. The guide says Python should work from the command line and recommends installing `setuptools` if needed. ([GitHub][1])

In our case, `npm install` succeeded, so Python was already good enough.

Optional command:

```powershell
python -m pip install --upgrade pip setuptools
```

Only do this if a future `node-gyp` error complains about Python or missing Python utilities.

---

# 5. Force node-gyp to use Visual Studio 2022

We tried:

```powershell
npm config set msvs_version 2022
```

But your npm gave:

```text
`msvs_version` is not a valid npm option
```

So the reliable PowerShell method is to set environment variables in the same terminal session:

```powershell
$env:GYP_MSVS_VERSION="2022"
$env:npm_config_msvs_version="2022"
```

Verify:

```powershell
echo $env:GYP_MSVS_VERSION
echo $env:npm_config_msvs_version
```

Expected:

```text
2022
2022
```

Alternative official method:

```powershell
npm config edit
```

Then manually add:

```ini
msvs_version=2022
```

The VS Code guide mentions using `npm config edit` to set `msvs_version=2022`. ([GitHub][1])

---

# 6. Clean the failed install

Because the first `npm install` failed midway, we cleaned the broken `node_modules`.

Close VS Code and all terminals using the repo, then open a fresh PowerShell.

```powershell
cd C:\Users\anura\learning-locally\vscode

taskkill /F /IM node.exe

Remove-Item -Recurse -Force .\node_modules -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "$env:USERPROFILE\AppData\Local\node-gyp" -ErrorAction SilentlyContinue
```

If `node_modules` does not delete because files are locked, restart Windows once and run the cleanup again.

For a fresh clone, this is also okay:

```powershell
git clean -xfd
```

Warning: `git clean -xfd` deletes all untracked files. Use it only when you do not have personal untracked files inside the repo.

The VS Code guide also recommends deleting the node-gyp cache and running `git clean -xfd` when native module builds fail. ([GitHub][1])

---

# 7. Install VS Code repo dependencies

From the repo root:

```powershell
cd C:\Users\anura\learning-locally\vscode

nvm use 24.17.0

$env:GYP_MSVS_VERSION="2022"
$env:npm_config_msvs_version="2022"

npm install
```

This is required. The official guide says to install and build dependencies using `npm install`. ([GitHub][1])

Ignore warnings like these unless the install actually fails:

```text
Unknown project config "disturl"
Unknown project config "target"
Unknown project config "runtime"
deprecated ...
```

Those warnings were not the main problem. The real earlier problem was the missing Spectre libraries and Node version mismatch.

---

# 8. Build VS Code source

After `npm install` succeeds:

```powershell
npm run watch
```

This does the initial full build and then keeps watching for changes. Wait until you see a message containing:

```text
Finished compilation
```

The VS Code guide says `npm run watch` runs the core watch task and extension watch tasks, does an initial full build, then watches for incremental changes. ([GitHub][1])

---

# 9. Launch local development VS Code

Keep `npm run watch` running in one terminal.

Open a second PowerShell:

```powershell
cd C:\Users\anura\learning-locally\vscode
.\scripts\code.bat
```

This launches the local dev build as:

```text
Code OSS Dev
```

The official Windows command is `.\scripts\code.bat`. The guide also says if you get “not a valid Electron app”, it usually means `npm run watch` was not run first. ([GitHub][1])

For macOS or Linux:

```bash
./scripts/code.sh
```

---

# 10. Stop watch before benchmarking

For CLI benchmarking, stop background work so timings are cleaner.

In the terminal running watch:

```text
Ctrl + C
```

Also close the Code OSS Dev window before measuring pure CLI type-check performance.

---

# 11. Open repo in normal VS Code

To open the source repo in your regular installed VS Code:

```powershell
code C:\Users\anura\learning-locally\vscode
```

Or:

```powershell
cd C:\Users\anura\learning-locally\vscode
code .
```

Do not use `.\scripts\code.bat` for this. That opens the local development build.

---

# 12. Install TypeScript 7 and TypeScript 6 globally

## Important mistake we found

This command caused confusion:

```powershell
npm install -g typescript@npm:@typescript/typescript6
```

This installs `@typescript/typescript6` under the package name `typescript`. So it replaces the normal TypeScript package globally.

That is why this happened:

```text
Install TS6 alias globally -> tsc disappeared, tsc6 appeared
Install normal typescript globally -> tsc6 disappeared, tsc appeared
```

Microsoft’s TypeScript 7 announcement says `@typescript/typescript6` provides the `tsc6` executable so TypeScript 6 and 7 can run side-by-side, and also explains npm alias behavior. ([Microsoft for Developers][3])

## Clean global install

Run:

```powershell
npm uninstall -g typescript @typescript/typescript6
```

Now install TypeScript 7 normally:

```powershell
npm install -g typescript
```

Install TypeScript 6 compatibility package separately:

```powershell
npm install -g @typescript/typescript6
```

Check:

```powershell
tsc -v
tsc6 -v
```

Expected:

```text
tsc  -> Version 7.x.x
tsc6 -> Version 6.x.x
```

If global `tsc` or `tsc6` is still not recognized, use `npx` instead. For benchmarking, `npx` is safer because it avoids global PATH issues.

---

# 13. Recommended non-global benchmark commands

This is the cleanest way.

TypeScript 7:

```powershell
npx tsc -v
npx tsc --noEmit -p ./src/tsconfig.json --extendedDiagnostics
```

TypeScript 6:

```powershell
npx -p @typescript/typescript6 tsc6 -v
$env:NODE_OPTIONS="--max-old-space-size=8192"
npx -p @typescript/typescript6 tsc6 --noEmit -p ./src/tsconfig.json --extendedDiagnostics
Remove-Item Env:NODE_OPTIONS
```

---

# 14. PowerShell syntax for NODE_OPTIONS

This does not work in PowerShell:

```powershell
NODE_OPTIONS="--max-old-space-size=8192" tsc6 --noEmit -p ./src/tsconfig.json --extendedDiagnostics
```

That syntax is for Bash.

Correct PowerShell syntax:

```powershell
$env:NODE_OPTIONS="--max-old-space-size=8192"
tsc6 --noEmit -p ./src/tsconfig.json --extendedDiagnostics
Remove-Item Env:NODE_OPTIONS
```

One-line version:

```powershell
$env:NODE_OPTIONS="--max-old-space-size=8192"; tsc6 --noEmit -p ./src/tsconfig.json --extendedDiagnostics; Remove-Item Env:NODE_OPTIONS
```

With `npx`:

```powershell
$env:NODE_OPTIONS="--max-old-space-size=8192"
npx -p @typescript/typescript6 tsc6 --noEmit -p ./src/tsconfig.json --extendedDiagnostics
Remove-Item Env:NODE_OPTIONS
```

If TypeScript 6 still runs out of memory, use 12 GB:

```powershell
$env:NODE_OPTIONS="--max-old-space-size=12288"
tsc6 --noEmit -p ./src/tsconfig.json --extendedDiagnostics
Remove-Item Env:NODE_OPTIONS
```

---

# 15. TypeScript 7 benchmark command

Using global `tsc`:

```powershell
tsc --noEmit -p ./src/tsconfig.json --extendedDiagnostics
```

Using local or npx:

```powershell
npx tsc --noEmit -p ./src/tsconfig.json --extendedDiagnostics
```

---

# 16. TypeScript 6 benchmark command

Using global `tsc6`:

```powershell
$env:NODE_OPTIONS="--max-old-space-size=8192"
tsc6 --noEmit -p ./src/tsconfig.json --extendedDiagnostics
Remove-Item Env:NODE_OPTIONS
```

Using `npx`:

```powershell
$env:NODE_OPTIONS="--max-old-space-size=8192"
npx -p @typescript/typescript6 tsc6 --noEmit -p ./src/tsconfig.json --extendedDiagnostics
Remove-Item Env:NODE_OPTIONS
```

---

# 17. Why TypeScript 6 needed NODE_OPTIONS

Without increasing Node heap, TypeScript 6 crashed with:

```text
FATAL ERROR: Ineffective mark-compacts near heap limit
Allocation failed - JavaScript heap out of memory
```

TypeScript 6 runs through Node.js and V8, so it can hit the default heap limit on a huge repo like VS Code.

TypeScript 7 native handled the same project much better because it is no longer the old JavaScript-based compiler. The TypeScript 7 announcement says TypeScript 7 uses native-code speed, shared-memory multithreading, and parallelization. ([Microsoft for Developers][3])

---

# 18. Commands for repeated benchmarking

For better benchmark quality, run each command multiple times.

## TypeScript 7 repeated test

```powershell
1..3 | ForEach-Object {
  tsc --noEmit -p ./src/tsconfig.json --extendedDiagnostics
}
```

This prints full diagnostics each time.

For easier manual recording, run one by one:

```powershell
tsc --noEmit -p ./src/tsconfig.json --extendedDiagnostics
tsc --noEmit -p ./src/tsconfig.json --extendedDiagnostics
tsc --noEmit -p ./src/tsconfig.json --extendedDiagnostics
```

# 19. Editor performance testing

For editor performance, CLI timing is not enough. Test these actions:

```text
cold project loading
autocomplete
hover
go to definition
find all references
rename symbol
```

The TypeScript 7 announcement says editor support is based on LSP and can leverage multiple threads to serve simultaneous requests quickly. It also says the TypeScript 7 language server can be enabled or disabled from the VS Code command palette. ([Microsoft for Developers][3])

## Good file to open

```text
src/vs/workbench/browser/workbench.ts
```

Good symbols to test:

```text
Workbench
Disposable
IInstantiationService
IConfigurationService
ILifecycleService
IWorkbenchLayoutService
```

## Another heavy file

```text
src/vs/editor/common/model/textModel.ts
```

Good symbols:

```text
TextModel
ITextModel
ITextBuffer
```

## Suggested editor test flow

1. Close all VS Code windows.
2. Open the VS Code repo.
3. Open `src/vs/workbench/browser/workbench.ts`.
4. Start timer.
5. Wait until autocomplete, hover, and errors become available.
6. Hover on `Workbench`.
7. Go to definition on `Disposable`.
8. Find all references on `Workbench`.
9. Repeat with TypeScript 6 language server.
10. Repeat with TypeScript 7 language server.

## Commands in VS Code

Open Command Palette:

```text
Ctrl + Shift + P
```

Useful commands:

```text
Enable TypeScript 7 Language Server
Disable TypeScript 7 Language Server
TypeScript: Open TS Server Log
Developer: Set Log Level
Developer: Open Logs Folder
```

---

# 20. Editor performance explanation

In our testing, TypeScript 7 editor performance looked much faster than CLI type-checking. Sometimes editor readiness, suggestions, and find-all-references felt around 50x to 60x faster.

This can happen because editor performance measures more than pure type-checking. It includes:

```text
language server startup
project loading
module resolution
symbol indexing
autocomplete preparation
hover response
find-all-references response
go-to-definition response
```

TypeScript 7 uses a new native implementation, LSP-based language tooling, shared-memory multithreading, and parallel processing. Microsoft also says TypeScript 7’s language server reduced failing language-server commands by over 80% and server crashes by over 60% compared with TypeScript 6. ([Microsoft for Developers][3])

Use this exact sequence from a fresh setup.

```powershell
cd C:\Users\anura\learning-locally

git clone https://github.com/microsoft/vscode.git
cd vscode

nvm install 24.17.0
nvm use 24.17.0
node -v

$env:GYP_MSVS_VERSION="2022"
$env:npm_config_msvs_version="2022"

npm install

npm run watch
```

In another terminal:

```powershell
cd C:\Users\anura\learning-locally\vscode
.\scripts\code.bat
```

For normal VS Code:

```powershell
code C:\Users\anura\learning-locally\vscode
```

Install global compilers:

```powershell
npm uninstall -g typescript @typescript/typescript6

npm install -g typescript
npm install -g @typescript/typescript6

tsc -v
tsc6 -v
```

TypeScript 7 benchmark:

```powershell
tsc --noEmit -p ./src/tsconfig.json --extendedDiagnostics
```

TypeScript 6 benchmark:

```powershell
$env:NODE_OPTIONS="--max-old-space-size=8192"
tsc6 --noEmit -p ./src/tsconfig.json --extendedDiagnostics
Remove-Item Env:NODE_OPTIONS
```

Safer `npx` version:

```powershell
npx tsc --noEmit -p ./src/tsconfig.json --extendedDiagnostics

$env:NODE_OPTIONS="--max-old-space-size=8192"
npx -p @typescript/typescript6 tsc6 --noEmit -p ./src/tsconfig.json --extendedDiagnostics
Remove-Item Env:NODE_OPTIONS
```

---

[1]: https://raw.githubusercontent.com/wiki/microsoft/vscode/How-to-Contribute.md "raw.githubusercontent.com"
[2]: https://raw.githubusercontent.com/microsoft/vscode/main/.nvmrc "raw.githubusercontent.com"
[3]: https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/ "Announcing TypeScript 7.0 - TypeScript"
