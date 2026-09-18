# Universal IDE
The repository should be designed so local-only operation works first, with no account, API key, registration, or external network required.

Sherin Dev Console — Universal IDE

<img width="759" height="2078" alt="image" src="https://github.com/user-attachments/assets/4416020d-c293-4e18-8567-f3b6ff7c7f7b" />



1. Core architecture
   

The key pipeline should be locked as:

                  SHERIN DEV CONSOLE
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
    File/Editor      Browser/Model      Terminal
        │                │
        └────────┬───────┘
                 ▼
            Input Router
                 │
                 ▼
             Kulexicon
                 │
       ┌─────────┴─────────┐
       │                   │
   ordinary text      execution block
       │                   │
       │                   ▼
       │          Execution Candidate
       │                   │
       │                   ▼
       │            Capability Request
       │                   │
       └───────────────────┤
                           ▼
                       SDRP Engine
                           │
             Capability → Policy → Trust
                           │
                        Select
                           │
                       Transport
                           │
                        Execute
                           │
                           ▼
                      Local Route
                           │
                           ▼
                       Executor

That keeps the responsibilities clean.

2. SDRP is the authority

The locked rule becomes a hard architectural invariant:

REQUESTER
    ↓
capability
    ↓
SDRP
    ↓
authorized route

Never:

POST /api/something
Authorization: Bearer ...

for internal capability routing.

For example:

terminal.execute
file.read
file.write
browser.open
tts.speak
model.infer
device.discover

are capabilities.

SDRP resolves them.

3. Kulexicon is the universal execution-language layer

This is where your latest idea belongs.

Kulexicon should understand execution documents from multiple environments:

PowerShell
CMD / BAT
Bash
Zsh
Fish
WSL
Linux
macOS
Python
Node
npm
pnpm
Yarn
Bun
Cargo
Git
Docker
CMake
Make
etc.

But it should not execute them.

Its job is:

```
TEXT
 ↓
Fence detection
 ↓
Language detection
 ↓
Lexical recognition
 ↓
KU representation
 ↓
ExecutionCandidate
```
Example:

```
powershell
Get-ChildItem
npm run build
```

becomes conceptually:

```
text
ExecutionCandidate {
    language: "powershell",
    commands: [
        "Get-ChildItem",
        "npm run build"
    ],
    source: "model-paste"
}
```

Then:
```
ExecutionCandidate
        ↓
CapabilityRequest
        ↓
terminal.execute
        ↓
SDRP
```

4. Universal terminal abstraction

The IDE should not care whether the machine is Windows, Linux or macOS.

Define:
```
TerminalAdapter

detect()
identity()
capabilities()
execute()
cancel()
status()
```
Then:
```
PowerShellAdapter
CmdAdapter
BashAdapter
WslAdapter
ZshAdapter
MacOSAdapter
LinuxAdapter
```
can all implement the same interface.

So the IDE sees:
```
terminal.execute
```
rather than:
```
powershell.exe
cmd.exe
bash
wsl.exe
zsh
```
SDRP resolves the appropriate local route.

5. Universal mode

This gives the IDE a proper Universal Mode.

The user/model can provide:
```
Get-ChildItem
```
or:

dir

or:
```
ls -la
```
or:
```
wsl ls -la
```
and the system determines:
```
language
       ↓
execution environment
       ↓
local capability
       ↓
SDRP route
```
The important part is that recognition is universal, execution is environment-aware.

6. SDRP startup

Your existing acceptance test becomes the first boot contract:
```
IDE starts
   ↓
SdrpEngine.bootstrap()
   ↓
LOCAL_BOOTSTRAPPING
   ↓
local capability discovery
   ↓
LOCAL_READY
```
No:
```
account
API key
registration
remote dependency
external network
```
Then Diagnostics should expose something like:
```
SDRP DIAGNOSTICS

State
────────────────────
LOCAL_READY

Node
────────────────────
Local Node
Trust: LOCAL

Capabilities
────────────────────
✓ terminal.execute
✓ file.read
✓ file.write
✓ browser.open

Routes
────────────────────
terminal.execute
  → LOCAL
  → PowerShell
  → READY

file.read
  → LOCAL
  → filesystem
  → READY
```
That becomes incredibly useful when debugging the IDE.

7. Security boundary

I would make this non-negotiable:
```
Kulexicon
   ↓
candidate

NOT

Kulexicon
   ↓
execute
```
The only execution authority is:
```
SDRP
 ↓
Policy
 ↓
Executor
```
So even if a model produces:
```
rm -rf ...
```
or:
```
Remove-Item ...
```
Kulexicon merely recognizes it.

SDRP/Policy decides what happens.

That gives you the fast path without creating an execution bypass.

8. Autonomous agent comes later

Then the local Ollama agent plugs into exactly the same interface:
```
Ollama
   ↓
Model response
   ↓
Input Router
   ↓
Kulexicon
   ↓
Capability Request
   ↓
SDRP
   ↓
Policy
   ↓
Terminal
   ↓
Result
   ↓
Context
   ↓
Ollama
```
So there is one execution architecture whether the input came from:

a human,
ChatGPT,
Grok,
Gemini,
Claude,
Ollama,
another local model,
a file,
or the IDE itself.

That's the important part.

Repository identity

I'd use:
```
Repository:
Sherin-Dev-Console

Product:
Sherin Dev Console

Protocol:
SDRP — Sherin Device/Service Routing Protocol

Execution layer:
Kulexicon

Agent layer:
Sherin Agent / Local Agent

Primary principle:
Zero Payload · Local First · Capability Routed
```
And the architectural slogan:
```
ANY MODEL
    ↓
ANY EXECUTION LANGUAGE
    ↓
KULEXICON
    ↓
ANY LOCAL PLATFORM
    ↓
SDRP
    ↓
AUTHORIZED CAPABILITY
```
That is the universal IDE foundation. It also means the current Browser / Model Surface, your SDRP v0.1 work, Kulexicon, Policy gate, Terminal, and future Ollama loop all become parts of one coherent repository instead of separate experiments.
