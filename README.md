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

The exact error we fixed was:

```text
error MSB8040: Spectre-mitigated libraries are required for this project
```

The VS Code guide says this error is fixed by installing the MSVC Spectre-mitigated libs, ATL Spectre, and MFC Spectre components. ([GitHub][1])

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

Your TypeScript 7 result was:

```text
Files:              8296
Lines:           2624546
Identifiers:     4235299
Symbols:         7021100
Types:           2468306
Instantiations:  3689476
Memory used:    4745629K
Memory allocs:  26906194
Config time:      0.318s
Parse time:       2.304s
Bind time:        0.265s
Check time:      15.338s
Emit time:        0.179s
Total time:      18.425s
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

Your TypeScript 6 result was:

```text
Files:                         8298
Lines of Library:             57729
Lines of Definitions:        314263
Lines of TypeScript:        2251430
Lines of JavaScript:              0
Lines of JSON:                 1559
Lines of Other:                   0
Identifiers:                4239075
Symbols:                    5689665
Types:                      1680036
Instantiations:             2357431
Memory used:               5500603K
Assignability cache size:    576988
Identity cache size:          25193
Subtype cache size:          146701
Strict subtype cache size:   114634
I/O Read time:                2.04s
Parse time:                   9.88s
ResolveModule time:           2.54s
ResolveTypeReference time:    0.02s
Program time:                16.87s
Bind time:                    6.07s
Check time:                  88.95s
printTime time:               0.00s
Emit time:                    0.00s
Total time:                 111.89s
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

# 18. Final benchmark report from our run

## Raw data

| Metric      | TypeScript 7 native | TypeScript 6 |
| ----------- | ------------------: | -----------: |
| Files       |               8,296 |        8,298 |
| Total time  |             18.425s |      111.89s |
| Check time  |             15.338s |       88.95s |
| Parse time  |              2.304s |        9.88s |
| Bind time   |              0.265s |        6.07s |
| Memory used |          4,745,629K |   5,500,603K |

The file count is slightly different, so it is not 100% identical internally, but it is close enough for a practical benchmark on the VS Code repo.

## Speedup

Total time:

```text
111.89 / 18.425 = 6.07x faster
```

So:

```text
TypeScript 7 was around 6.07x faster than TypeScript 6.
```

Time reduction:

```text
111.89 - 18.425 = 93.465 seconds saved
```

Percentage reduction:

```text
93.465 / 111.89 * 100 = 83.53%
```

So:

```text
TypeScript 7 reduced total type-checking time by around 83.5%.
```

## Check-time speedup

```text
88.95 / 15.338 = 5.80x faster
```

So:

```text
The check phase alone was around 5.8x faster.
```

Check-time reduction:

```text
88.95 - 15.338 = 73.612 seconds saved
```

Percentage reduction:

```text
73.612 / 88.95 * 100 = 82.76%
```

So:

```text
TypeScript 7 reduced check time by around 82.8%.
```

## Memory comparison

TypeScript 7:

```text
4,745,629K
```

TypeScript 6:

```text
5,500,603K
```

Difference:

```text
5,500,603K - 4,745,629K = 754,974K
```

That is approximately:

```text
737 MB less memory
0.72 GiB less memory
```

Percentage less memory:

```text
754,974 / 5,500,603 * 100 = 13.73%
```

So:

```text
TypeScript 7 used around 13.7% less memory.
```

Memory ratio:

```text
5,500,603 / 4,745,629 = 1.16x
```

So:

```text
TypeScript 6 used around 1.16x more memory than TypeScript 7.
```

---

# 19. Commands for repeated benchmarking

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

## TypeScript 6 repeated test

```powershell
$env:NODE_OPTIONS="--max-old-space-size=8192"

tsc6 --noEmit -p ./src/tsconfig.json --extendedDiagnostics
tsc6 --noEmit -p ./src/tsconfig.json --extendedDiagnostics
tsc6 --noEmit -p ./src/tsconfig.json --extendedDiagnostics

Remove-Item Env:NODE_OPTIONS
```

---

# 20. Optional TypeScript 7 parallelization tests

TypeScript 7 supports these performance-related flags:

```text
--checkers
--builders
--singleThreaded
```

The TypeScript 7 announcement says TypeScript 7 performs many steps in parallel, including parsing, type-checking, and emitting. It also says the default number of type-checking workers is 4 and can be changed with `--checkers`. ([Microsoft for Developers][3])

Try:

```powershell
tsc --noEmit -p ./src/tsconfig.json --extendedDiagnostics --checkers 1
tsc --noEmit -p ./src/tsconfig.json --extendedDiagnostics --checkers 4
tsc --noEmit -p ./src/tsconfig.json --extendedDiagnostics --checkers 8
```

Single-threaded test:

```powershell
tsc --noEmit -p ./src/tsconfig.json --extendedDiagnostics --singleThreaded
```

Use this to show how much TypeScript 7 benefits from concurrency and multithreading.

---

# 21. Editor performance testing

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

For manual video demo, a screen recording with timer is easiest.

---

# 22. Editor performance explanation

In our testing, TypeScript 7 editor performance looked much faster than CLI type-checking. Sometimes editor readiness, suggestions, and find-all-references felt around 40x to 60x faster.

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

Use this careful wording:

```text
In CLI type-checking, TypeScript 7 was around 6x faster on my machine.
But in editor responsiveness, especially cold loading, autocomplete, hover, and find-all-references, the perceived improvement was sometimes much higher, around 40x to 60x in my manual testing.
```

Do not say globally:

```text
TypeScript 7 is always 60x faster.
```

Better:

```text
In my editor performance test on the VS Code repo, some user-visible operations felt up to 40x to 60x faster.
```

---

# 23. Final command checklist

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
