# <center>LLVM opt 使用文档</center>
```
OVERVIEW: llvm .bc -> .bc modular optimizer and analysis printer

USAGE: opt [options] <input bitcode file>

OPTIONS:

Color Options:

  --color                                                               - Use colors in output (default=autodetect)

General options:

  --O0                                                                  - Optimization level 0. Similar to clang -O0. Same as -passes="default<O0>"
  --O1                                                                  - Optimization level 1. Similar to clang -O1. Same as -passes="default<O1>"
  --O2                                                                  - Optimization level 2. Similar to clang -O2. Same as -passes="default<O2>"
  --O3                                                                  - Optimization level 3. Similar to clang -O3. Same as -passes="default<O3>"
  --Os                                                                  - Like -O2 but size-conscious. Similar to clang -Os. Same as -passes="default<Os>"
  --Oz                                                                  - Like -O2 but optimize for code size above all else. Similar to clang -Oz. Same as -passes="default<Oz>"
  -S                                                                    - Write output as LLVM assembly
  --abort-on-max-devirt-iterations-reached                              - Abort when the max iterations for devirtualization CGSCC repeat pass is reached
  --addrsig                                                             - Emit an address-significance table
  --align-loops=<uint>                                                  - Default alignment for loops
  --allow-ginsert-as-artifact                                           - Allow G_INSERT to be considered an artifact. Hack around AMDGPU test infinite loops.
  --asm-show-inst                                                       - Emit internal instruction representation to assembly file
  --atomic-counter-update-promoted                                      - Do counter update using atomic fetch add  for promoted counters only
  --atomic-first-counter                                                - Use atomic fetch add for first counter in a function (usually the entry counter)
  --basic-block-address-map                                             - Emit the basic block address map section
  --basic-block-sections=<all | <function list (file)> | labels | none> - Emit basic blocks into separate sections
  --bounds-checking-single-trap                                         - Use one trap block per function
  --bugpoint-enable-legacy-pm                                           - Enable the legacy pass manager. This is strictly for bugpoint due to it not working with the new PM, please do not use otherwise.
  --cfg-hide-cold-paths=<number>                                        - Hide blocks with relative frequency below the given value
  --cfg-hide-deoptimize-paths                                           - 
  --cfg-hide-unreachable-paths                                          - 
  --check-functions-filter=<regex>                                      - Only emit checks for arguments of functions whose names match the given regular expression
  --code-model=<value>                                                  - Choose code model
    =tiny                                                               -   Tiny code model
    =small                                                              -   Small code model
    =kernel                                                             -   Kernel code model
    =medium                                                             -   Medium code model
    =large                                                              -   Large code model
  --codegen-opt-level=<uint>                                            - Override optimization level for codegen hooks, legacy PM only
  --conditional-counter-update                                          - Do conditional counter updates in single byte counters mode)
  --cost-kind=<value>                                                   - Target cost kind
    =throughput                                                         -   Reciprocal throughput
    =latency                                                            -   Instruction latency
    =code-size                                                          -   Code size
    =size-latency                                                       -   Code size and latency
  --crel                                                                - Use CREL relocation format for ELF
  --data-layout=<layout-string>                                         - data layout string to use
  --data-sections                                                       - Emit data into separate sections
  --debug-entry-values                                                  - Enable debug info for the debug entry values.
  --debug-info-correlate                                                - Use debug info to correlate profiles. (Deprecated, use -profile-correlate=debug-info)
  --debugger-tune=<value>                                               - Tune debug info for a particular debugger
    =gdb                                                                -   gdb
    =lldb                                                               -   lldb
    =dbx                                                                -   dbx
    =sce                                                                -   SCE targets (e.g. PS4)
  --debugify-each                                                       - Start each pass with debugify and end it with check-debugify
  --debugify-export=<filename>                                          - Export per-pass debugify statistics to this file
  --debugify-func-limit=<ulong>                                         - Set max number of processed functions per pass.
  --debugify-level=<value>                                              - Kind of debug info to add
    =locations                                                          -   Locations only
    =location+variables                                                 -   Locations and Variables
  --debugify-quiet                                                      - Suppress verbose debugify output
  --denormal-fp-math=<value>                                            - Select which denormal numbers the code is permitted to require
    =ieee                                                               -   IEEE 754 denormal numbers
    =preserve-sign                                                      -   the sign of a  flushed-to-zero number is preserved in the sign of 0
    =positive-zero                                                      -   denormals are flushed to positive zero
    =dynamic                                                            -   denormals have unknown treatment
  --denormal-fp-math-f32=<value>                                        - Select which denormal numbers the code is permitted to require for float
    =ieee                                                               -   IEEE 754 denormal numbers
    =preserve-sign                                                      -   the sign of a  flushed-to-zero number is preserved in the sign of 0
    =positive-zero                                                      -   denormals are flushed to positive zero
    =dynamic                                                            -   denormals have unknown treatment
  --disable-auto-upgrade-debug-info                                     - Disable autoupgrade of debug info
  --disable-builtin=<string>                                            - Disable specific target library builtin function
  --disable-debug-info-type-map                                         - Don't use a uniquing type map for debug info
  --disable-i2p-p2i-opt                                                 - Disables inttoptr/ptrtoint roundtrip optimization
  --disable-loop-unrolling                                              - Disable loop unrolling in all relevant passes
  --disable-simplify-libcalls                                           - Disable simplify-libcalls
  --disable-tail-calls                                                  - Never emit tail calls
  --do-counter-promotion                                                - Do counter register promotion
  --dot-cfg-mssa=<file name for generated dot file>                     - file name for generated dot file
  --dwarf-version=<int>                                                 - Dwarf version
  --dwarf64                                                             - Generate debugging info in the 64-bit DWARF format
  --emit-call-site-info                                                 - Emit call site debug information, if debug information is enabled.
  --emit-compact-unwind-non-canonical                                   - Whether to try to emit Compact Unwind for non canonical entries.
  --emit-dwarf-unwind=<value>                                           - Whether to emit DWARF EH frame entries.
    =always                                                             -   Always emit EH frame entries
    =no-compact-unwind                                                  -   Only emit EH frame entries when compact unwind is not available
    =default                                                            -   Use target platform default
  --emulated-tls                                                        - Use emulated TLS model
  --enable-approx-func-fp-math                                          - Enable FP math optimizations that assume approx func
  --enable-cse-in-irtranslator                                          - Should enable CSE in irtranslator
  --enable-cse-in-legalizer                                             - Should enable CSE in Legalizer
  --enable-debugify                                                     - Start the pipeline with debugify and end it with check-debugify
  --enable-gvn-hoist                                                    - Enable the GVN hoisting pass (default = off)
  --enable-gvn-memdep                                                   - 
  --enable-gvn-memoryssa                                                - 
  --enable-gvn-sink                                                     - Enable the GVN sinking pass (default = off)
  --enable-jmc-instrument                                               - Instrument functions with a call to __CheckForDebuggerJustMyCode
  --enable-jump-table-to-switch                                         - Enable JumpTableToSwitch pass (default = off)
  --enable-load-in-loop-pre                                             - 
  --enable-load-pre                                                     - 
  --enable-loop-simplifycfg-term-folding                                - 
  --enable-name-compression                                             - Enable name/filename string compression
  --enable-no-infs-fp-math                                              - Enable FP math optimizations that assume no +-Infs
  --enable-no-nans-fp-math                                              - Enable FP math optimizations that assume no NaNs
  --enable-no-signed-zeros-fp-math                                      - Enable FP math optimizations that assume the sign of 0 is insignificant
  --enable-no-trapping-fp-math                                          - Enable setting the FP exceptions build attribute not to use exceptions
  --enable-split-backedge-in-load-pre                                   - 
  --enable-split-loopiv-heuristic                                       - Enable loop iv regalloc heuristic
  --enable-tlsdesc                                                      - Enable the use of TLS Descriptors
  --enable-unsafe-fp-math                                               - Enable optimizations that may decrease FP precision
  --enable-vtable-profile-use                                           - If ThinLTO and WPD is enabled and this option is true, vtable profiles will be used by ICP pass for more efficient indirect call sequence. If false, type profiles won't be used.
  --enable-vtable-value-profiling                                       - If true, the virtual table address will be instrumented to know the types of a C++ pointer. The information is used in indirect call promotion to do selective vtable-based comparison.
  --exception-model=<value>                                             - exception model
    =default                                                            -   default exception handling model
    =dwarf                                                              -   DWARF-like CFI based exception handling
    =sjlj                                                               -   SjLj exception handling
    =arm                                                                -   ARM EHABI exceptions
    =wineh                                                              -   Windows exception model
    =wasm                                                               -   WebAssembly exception handling
  --expand-variadics-override=<value>                                   - Override the behaviour of expand-variadics
    =unspecified                                                        -   Use the implementation defaults
    =disable                                                            -   Disable the pass entirely
    =optimize                                                           -   Optimise without changing ABI
    =lowering                                                           -   Change variadic calling convention
  --experimental-debug-variable-locations                               - Use experimental new value-tracking variable locations
  --experimental-debuginfo-iterators                                    - Enable communicating debuginfo positions through iterators, eliminating intrinsics. Has no effect if --preserve-input-debuginfo-format=true.
  -f                                                                    - Enable binary output on terminals
  --fatal-warnings                                                      - Treat warnings as errors
  --fdpic                                                               - Use the FDPIC ABI
  --filetype=<value>                                                    - Choose a file type (not all types are supported by all targets):
    =asm                                                                -   Emit an assembly ('.s') file
    =obj                                                                -   Emit a native object ('.o') file
    =null                                                               -   Emit nothing, for performance testing
  --float-abi=<value>                                                   - Choose float ABI type
    =default                                                            -   Target default float ABI type
    =soft                                                               -   Soft float ABI (implied by -soft-float)
    =hard                                                               -   Hard float ABI (uses FP registers)
  --force-dwarf-frame-section                                           - Always emit a debug frame section.
  --force-tail-folding-style=<value>                                    - Force the tail folding style
    =none                                                               -   Disable tail folding
    =data                                                               -   Create lane mask for data only, using active.lane.mask intrinsic
    =data-without-lane-mask                                             -   Create lane mask with compare/stepvector
    =data-and-control                                                   -   Create lane mask using active.lane.mask intrinsic, and use it for both data and control flow
    =data-and-control-without-rt-check                                  -   Similar to data-and-control, but remove the runtime check
    =data-with-evl                                                      -   Use predicated EVL instructions for tail folding. If EVL is unsupported, fallback to data-without-lane-mask.
  --fp-contract=<value>                                                 - Enable aggressive formation of fused FP ops
    =fast                                                               -   Fuse FP ops whenever profitable
    =on                                                                 -   Only fuse 'blessed' FP ops.
    =off                                                                -   Only fuse FP ops when the result won't be affected.
  --frame-pointer=<value>                                               - Specify frame pointer elimination optimization
    =all                                                                -   Disable frame pointer elimination
    =non-leaf                                                           -   Disable frame pointer elimination for non-leaf frame
    =reserved                                                           -   Enable frame pointer elimination, but reserve the frame pointer register
    =none                                                               -   Enable frame pointer elimination
  --fs-profile-debug-bw-threshold=<uint>                                - Only show debug message if the source branch weight is greater  than this value.
  --fs-profile-debug-prob-diff-threshold=<uint>                         - Only show debug message if the branch probability is greater than this value (in percentage).
  --function-sections                                                   - Emit functions into separate sections
  --generate-merged-base-profiles                                       - When generating nested context-sensitive profiles, always generate extra base profile for function with all its context profiles merged into it.
  --hash-based-counter-split                                            - Rename counter variable of a comdat function based on cfg hash
  --hot-cold-split                                                      - Enable hot-cold splitting pass
  --hwasan-percentile-cutoff-hot=<int>                                  - Hot percentile cutoff.
  --hwasan-random-rate=<number>                                         - Probability value in the range [0.0, 1.0] to keep instrumentation of a function. Note: instrumentation can be skipped randomly OR because of the hot percentile cutoff, if both are supplied.
  --ignore-xcoff-visibility                                             - Not emit the visibility attribute for asm in AIX OS or give all symbols 'unspecified' visibility in XCOFF object file
  --implicit-mapsyms                                                    - Allow mapping symbol at section beginning to be implicit, lowering number of mapping symbols at the expense of some portability. Recommended for projects that can build all their object files using this option
  --import-all-index                                                    - Import all external functions in index.
  --incremental-linker-compatible                                       - When used with filetype=obj, emit an object file which can be used with an incremental linker
  --instcombine-code-sinking                                            - Enable code sinking
  --instcombine-guard-widening-window=<uint>                            - How wide an instruction window to bypass looking for another guard
  --instcombine-max-num-phis=<uint>                                     - Maximum number phis to handle in intptr/ptrint folding
  --instcombine-max-sink-users=<uint>                                   - Maximum number of undroppable users for instruction sinking
  --instcombine-maxarray-size=<uint>                                    - Maximum array size considered when doing a combine
  --instcombine-negator-enabled                                         - Should we attempt to sink negations?
  --instcombine-negator-max-depth=<uint>                                - What is the maximal lookup depth when trying to check for viability of negation sinking.
  --instrprof-atomic-counter-update-all                                 - Make all profile counter updates atomic (for testing only)
  --internalize-public-api-file=<filename>                              - A file containing list of symbol names to preserve
  --internalize-public-api-list=<list>                                  - A list of symbol names to preserve
  --iterative-counter-promotion                                         - Allow counter promotion across the whole loop nest.
  --large-data-threshold=<ulong>                                        - Choose large data threshold for x86_64 medium code model
  --lint-abort-on-error                                                 - In the Lint pass, abort on errors.
  --load=<pluginfilename>                                               - Load the specified plugin
  --load-pass-plugin=<string>                                           - Load passes from plugin library
  --lower-allow-check-percentile-cutoff-hot=<int>                       - Hot percentile cutoff.
  --lower-allow-check-random-rate=<number>                              - Probability value in the range [0.0, 1.0] of unconditional pseudo-random checks.
  --march=<string>                                                      - Architecture to generate code for (see --version)
  --matrix-default-layout=<value>                                       - Sets the default matrix layout
    =column-major                                                       -   Use column-major layout
    =row-major                                                          -   Use row-major layout
  --matrix-print-after-transpose-opt                                    - 
  --mattr=<a1,+a2,-a3,...>                                              - Target specific attributes (-mattr=help for details)
  --max-counter-promotions=<int>                                        - Max number of allowed counter promotions
  --max-counter-promotions-per-loop=<uint>                              - Max number counter promotions per loop to avoid increasing register pressure too much
  --mc-relax-all                                                        - When used with filetype=obj, relax all fixups in the emitted object file
  --mcpu=<cpu-name>                                                     - Target a specific cpu type (-mcpu=help for details)
  --meabi=<value>                                                       - Set EABI type (default depends on triple):
    =default                                                            -   Triple default EABI version
    =4                                                                  -   EABI version 4
    =5                                                                  -   EABI version 5
    =gnu                                                                -   EABI GNU
  --mir-strip-debugify-only                                             - Should mir-strip-debug only strip debug info from debugified modules by default
  --misexpect-tolerance=<uint>                                          - Prevents emitting diagnostics when profile counts are within N% of the threshold..
  --module-hash                                                         - Emit module hash
  --module-summary                                                      - Emit module summary index
  --mtriple=<string>                                                    - Override target triple for module
  --mxcoff-roptr                                                        - When set to true, const objects with relocatable address values are put into the RO data section.
  --no-deprecated-warn                                                  - Suppress all deprecated warnings
  --no-discriminators                                                   - Disable generation of discriminator information.
  --no-integrated-as                                                    - Disable integrated assembler
  --no-type-check                                                       - Suppress type errors (Wasm)
  --no-warn                                                             - Suppress all warnings
  --nozero-initialized-in-bss                                           - Don't place zero-initialized symbols into bss section
  -o <filename>                                                         - Override output filename
  --object-size-offset-visitor-max-visit-instructions=<uint>            - Maximum number of instructions for ObjectSizeOffsetVisitor to look at
  --partition-static-data-sections                                      - Partition data sections using profile information.
  --pass-remarks-filter=<regex>                                         - Only record optimization remarks from passes whose names match the given regular expression
  --pass-remarks-format=<format>                                        - The format used for serializing remarks (default: YAML)
  --pass-remarks-output=<filename>                                      - Output filename for pass remarks
  --passes=<string>                                                     - A textual description of the pass pipeline. To have analysis passes available before a certain pass, add "require<foo-analysis>".
  --pgo-block-coverage                                                  - Use this option to enable basic block coverage instrumentation
  --pgo-temporal-instrumentation                                        - Use this option to enable temporal instrumentation
  --pgo-view-block-coverage-graph                                       - Create a dot file of CFGs with block coverage inference information
  --print-passes                                                        - Print available passes that can be specified in -passes=foo and exit
  --print-pipeline-passes                                               - Print a '-passes' compatible string describing the pipeline (best-effort only).
  --profile-correlate=<value>                                           - Use debug info or binary file to correlate profiles.
    =<empty>                                                            -   No profile correlation
    =debug-info                                                         -   Use debug info to correlate
    =binary                                                             -   Use binary to correlate
  --relocation-model=<value>                                            - Choose relocation model
    =static                                                             -   Non-relocatable code
    =pic                                                                -   Fully relocatable, position independent code
    =dynamic-no-pic                                                     -   Relocatable external references, non-relocatable code
    =ropi                                                               -   Code and read-only data relocatable, accessed PC-relative
    =rwpi                                                               -   Read-write data relocatable, accessed relative to static base
    =ropi-rwpi                                                          -   Combination of ropi and rwpi
  --riscv-add-build-attributes                                          - 
  Optimizations available (use "-passes=" for the new pass manager)
      --aa                                                                 - Function Alias Analysis Results
      --always-inline                                                      - Inliner for always_inline functions
      --assumption-cache-tracker                                           - Assumption Cache Tracker
      --atomic-expand                                                      - Expand Atomic instructions
      --barrier                                                            - A No-Op Barrier Pass
      --basic-aa                                                           - Basic Alias Analysis (stateless AA impl)
      --basiccg                                                            - CallGraph Construction
      --bbsections-profile-reader                                          - Reads and parses a basic block sections profile.
      --block-freq                                                         - Block Frequency Analysis
      --branch-prob                                                        - Branch Probability Analysis
      --break-crit-edges                                                   - Break critical edges in CFG
      --callbrprepare                                                      - Prepare callbr
      --canon-freeze                                                       - Canonicalize Freeze Instructions in Loops
      --check-debugify                                                     - Check debug info from -debugify
      --check-debugify-function                                            - Check debug info from -debugify-function
      --codegenprepare                                                     - Optimize for code generation
      --consthoist                                                         - Constant Hoisting
      --cseinfo                                                            - Analysis containing CSE Info
      --cycles                                                             - Cycle Info Analysis
      --da                                                                 - Dependence Analysis
      --dce                                                                - Dead Code Elimination
      --deadargelim                                                        - Dead Argument Elimination
      --deadarghaX0r                                                       - Dead Argument Hacking (BUGPOINT USE ONLY; DO NOT USE)
      --debugify                                                           - Attach debug info to everything
      --debugify-function                                                  - Attach debug info to a function
      --domfrontier                                                        - Dominance Frontier Construction
      --domtree                                                            - Dominator Tree Construction
      --dot-callgraph                                                      - Print call graph to 'dot' file
      --dot-dom                                                            - Print dominance tree of function to 'dot' file
      --dot-dom-only                                                       - Print dominance tree of function to 'dot' file (with no function bodies)
      --dot-postdom                                                        - Print postdominance tree of function to 'dot' file
      --dot-postdom-only                                                   - Print postdominance tree of function to 'dot' file (with no function bodies)
      --dot-regions                                                        - Print regions of function to 'dot' file
      --dot-regions-only                                                   - Print regions of function to 'dot' file (with no function bodies)
      --dwarf-eh-prepare                                                   - Prepare DWARF exceptions
      --dxil-resource-binding                                              - DXIL Resource Binding Analysis
      --dxil-resource-type                                                 - DXIL Resource Type Analysis
      --early-cse                                                          - Early CSE
      --early-cse-memssa                                                   - Early CSE w/ MemorySSA
      --edge-bundles                                                       - Bundle Machine CFG Edges
      --expand-large-div-rem                                               - Expand large div/rem
      --expand-large-fp-convert                                            - Expand large fp convert
      --expand-memcmp                                                      - Expand memcmp() to load/stores
      --expand-reductions                                                  - Expand reduction intrinsics
      --external-aa                                                        - External Alias Analysis
      --fastpretileconfig                                                  - Fast Tile Register Preconfigure
      --fasttileconfig                                                     - Fast Tile Register Configure
      --fix-irreducible                                                    - Convert irreducible control-flow into natural loops
      --flattencfg                                                         - Flatten the CFG
      --gisel-known-bits                                                   - Analysis for ComputingKnownBits
      --global-merge                                                       - Merge global variables
      --globals-aa                                                         - Globals Alias Analysis
      --gvn                                                                - Global Value Numbering
      --indirectbr-expand                                                  - Expand indirectbr instructions
      --infer-address-spaces                                               - Infer address spaces
      --instcombine                                                        - Combine redundant instructions
      --instruction-select                                                 - Select target instructions out of generic instructions
      --instsimplify                                                       - Remove redundant instructions
      --interleaved-access                                                 - Lower interleaved memory accesses to target specific intrinsics
      --interleaved-load-combine                                           - Combine interleaved loads into wide loads and shufflevector instructions
      --ir-similarity-identifier                                           - ir-similarity-identifier
      --irtranslator                                                       - IRTranslator LLVM IR -> MI
      --iv-users                                                           - Induction Variable Users
      --jmc-instrumenter                                                   - Instrument function entry with call to __CheckForDebuggerJustMyCode
      --kcfi                                                               - Insert KCFI indirect call checks
      --lazy-block-freq                                                    - Lazy Block Frequency Analysis
      --lazy-branch-prob                                                   - Lazy Branch Probability Analysis
      --lazy-value-info                                                    - Lazy Value Information Analysis
      --lcssa                                                              - Loop-Closed SSA Form Pass
      --lcssa-verification                                                 - LCSSA Verifier
      --legalizer                                                          - Legalize the Machine IR a function's Machine IR
      --licm                                                               - Loop Invariant Code Motion
      --load-store-vectorizer                                              - Vectorize load and store instructions
      --loadstore-opt                                                      - Generic memory optimizations
      --localizer                                                          - Move/duplicate certain instructions close to their use
      --loop-data-prefetch                                                 - Loop Data Prefetch
      --loop-extract                                                       - Extract loops into new functions
      --loop-extract-single                                                - Extract at most one loop into a new function
      --loop-reduce                                                        - Loop Strength Reduction
      --loop-simplify                                                      - Canonicalize natural loops
      --loop-term-fold                                                     - Loop Terminator Folding
      --loop-unroll                                                        - Unroll loops
      --loops                                                              - Natural Loop Information
      --lower-amx-intrinsics                                               - Lower AMX intrinsics
      --lower-amx-type                                                     - Lower AMX type for load/store
      --lower-global-dtors                                                 - Lower @llvm.global_dtors via `__cxa_atexit`
      --loweratomic                                                        - Lower atomic intrinsics to non-atomic form
      --lowerinvoke                                                        - Lower invoke and unwind, for unwindless code generators
      --lowerswitch                                                        - Lower SwitchInst's to branches
      --lowertilecopy                                                      - Tile Copy Lowering
      --machine-block-freq                                                 - Machine Block Frequency Analysis
      --machine-branch-prob                                                - Machine Branch Probability Analysis
      --machine-domfrontier                                                - Machine Dominance Frontier Construction
      --machine-loops                                                      - Machine Natural Loop Construction
      --machinedomtree                                                     - MachineDominator Tree Construction
      --mem2reg                                                            - Promote Memory to Register
      --memdep                                                             - Memory Dependence Analysis
      --memoryssa                                                          - Memory SSA
      --mergeicmps                                                         - Merge contiguous icmps into a memcmp
      --module-summary-analysis                                            - Module Summary Analysis
      --module-summary-info                                                - Module summary info
      --nary-reassociate                                                   - Nary reassociation
      --opt-remark-emitter                                                 - Optimization Remark Emitter
      --partially-inline-libcalls                                          - Partially inline calls to library functions
      --phi-values                                                         - Phi Values Analysis
      --place-backedge-safepoints-impl                                     - Place Backedge Safepoints
      --post-inline-ee-instrument                                          - Instrument function entry/exit with calls to e.g. mcount() (post inlining)
      --postdomtree                                                        - Post-Dominator Tree Construction
      --pre-isel-intrinsic-lowering                                        - Pre-ISel Intrinsic Lowering
      --print-function                                                     - Print function to stderr
      --print-module                                                       - Print module to stderr
      --profile-summary-info                                               - Profile summary info
      --pseudo-probe-inserter                                              - Insert pseudo probe annotations for value profiling
      --reaching-defs-analysis                                             - ReachingDefAnalysis
      --reassociate                                                        - Reassociate expressions
      --regbankselect                                                      - Assign register bank of generic virtual registers
      --regions                                                            - Detect single entry single exit regions
      --replace-with-veclib                                                - Replace intrinsics with calls to vector library
      --riscv-O0-prelegalizer-combiner                                     - Combine RISC-V machine instrs before legalization
      --riscv-codegenprepare                                               - RISC-V CodeGenPrepare
      --riscv-dead-defs                                                    - RISC-V Dead register definitions
      --riscv-expand-pseudo                                                - RISC-V pseudo instruction expansion pass
      --riscv-fold-mem-offset                                              - RISC-V Fold Memory Offset
      --riscv-gather-scatter-lowering                                      - RISC-V gather/scatter lowering pass
      --riscv-insert-read-write-csr                                        - RISC-V Insert Read/Write CSR Pass
      --riscv-insert-vsetvli                                               - RISC-V Insert VSETVLI pass
      --riscv-insert-write-vxrm                                            - RISC-V Insert Write VXRM Pass
      --riscv-isel                                                         - RISC-V DAG->DAG Pattern Instruction Selection
      --riscv-make-compressible                                            - RISC-V Make Compressible
      --riscv-merge-base-offset                                            - RISC-V Merge Base Offset
      --riscv-move-merge                                                   - RISC-V Zcmp move merging pass
      --riscv-opt-w-instrs                                                 - RISC-V Optimize W Instructions
      --riscv-post-ra-expand-pseudo                                        - RISC-V post-regalloc pseudo instruction expansion pass
      --riscv-postlegalizer-combiner                                       - Combine RISC-V MachineInstrs after legalization
      --riscv-prelegalizer-combiner                                        - Combine RISC-V machine instrs before legalization
      --riscv-prera-expand-pseudo                                          - RISC-V Pre-RA pseudo instruction expansion pass
      --riscv-push-pop-opt                                                 - RISC-V Zcmp Push/Pop optimization pass
      --riscv-vector-peephole                                              - RISC-V Fold Masks
      --riscv-vl-optimizer                                                 - RISC-V VL Optimizer
      --riscv-vmv0-elimination                                             - RISC-V VMV0 Elimination
      --safe-stack                                                         - Safe Stack instrumentation pass
      --scalar-evolution                                                   - Scalar Evolution Analysis
      --scalarize-masked-mem-intrin                                        - Scalarize unsupported masked memory intrinsics
      --scalarizer                                                         - Scalarize vector operations
      --scev-aa                                                            - ScalarEvolution-based Alias Analysis
      --scoped-noalias-aa                                                  - Scoped NoAlias Alias Analysis
      --select-optimize                                                    - Optimize selects
      --separate-const-offset-from-gep                                     - Split GEPs to a variadic base and a constant offset for better CSE
      --simplifycfg                                                        - Simplify the CFG
      --sink                                                               - Code sinking
      --sjlj-eh-prepare                                                    - Prepare SjLj exceptions
      --slsr                                                               - Straight line strength reduction
      --speculative-execution                                              - Speculatively execute instructions
      --sroa                                                               - Scalar Replacement Of Aggregates
      --stack-protector                                                    - Insert stack protectors
      --stack-safety                                                       - Stack Safety Analysis
      --stack-safety-local                                                 - Stack Safety Local Analysis
      --structurizecfg                                                     - Structurize the CFG
      --tailcallelim                                                       - Tail Call Elimination
      --targetlibinfo                                                      - Target Library Information
      --targetpassconfig                                                   - Target Pass Configuration
      --tbaa                                                               - Type-Based Alias Analysis
      --tileconfig                                                         - Tile Register Configure
      --tilepreconfig                                                      - Tile Register Pre-configure
      --tti                                                                - Target Transform Information
      --uniformity                                                         - Uniformity Analysis
      --unify-loop-exits                                                   - Fixup each natural loop to have a single exit block
      --unreachableblockelim                                               - Remove unreachable blocks from the CFG
      --verify                                                             - Module Verifier
      --verify-safepoint-ir                                                - Safepoint IR Verifier
      --view-callgraph                                                     - View call graph
      --view-dom                                                           - View dominance tree of function
      --view-dom-only                                                      - View dominance tree of function (with no function bodies)
      --view-postdom                                                       - View postdominance tree of function
      --view-postdom-only                                                  - View postdominance tree of function (with no function bodies)
      --view-regions                                                       - View regions of function
      --view-regions-only                                                  - View regions of function (with no function bodies)
      --virtregmap                                                         - Virtual Register Map
      --wasm-eh-prepare                                                    - Prepare WebAssembly exceptions
      --win-eh-prepare                                                     - Prepare Windows exceptions
      --write-bitcode                                                      - Write Bitcode
      --x86-avoid-SFB                                                      - Machine code sinking
      --x86-avoid-trailing-call                                            - X86 avoid trailing call pass
      --x86-cf-opt                                                         - X86 Call Frame Optimization
      --x86-cmov-conversion                                                - X86 cmov Conversion
      --x86-codegen                                                        - X86 FP Stackifier
      --x86-compress-evex                                                  - Compressing EVEX instrs when possible
      --x86-domain-reassignment                                            - X86 Domain Reassignment Pass
      --x86-dyn-alloca-expander                                            - X86 DynAlloca Expander
      --x86-execution-domain-fix                                           - X86 Execution Domain Fix
      --x86-fixup-LEAs                                                     - X86 LEA Fixup
      --x86-fixup-bw-insts                                                 - X86 Byte/Word Instruction Fixup
      --x86-fixup-inst-tuning                                              - x86-fixup-inst-tuning
      --x86-fixup-setcc                                                    - x86-fixup-setcc
      --x86-fixup-vector-constants                                         - x86-fixup-vector-constants
      --x86-flags-copy-lowering                                            - X86 EFLAGS copy lowering
      --x86-isel                                                           - X86 DAG->DAG Instruction Selection
      --x86-lvi-load                                                       - X86 LVI load hardening
      --x86-lvi-ret                                                        - X86 LVI ret hardener
      --x86-optimize-LEAs                                                  - X86 optimize LEA pass
      --x86-partial-reduction                                              - X86 Partial Reduction
      --x86-pseudo                                                         - X86 pseudo instruction expansion pass
      --x86-return-thunks                                                  - X86 Return Thunks
      --x86-seses                                                          - X86 Speculative Execution Side Effect Suppression
      --x86-slh                                                            - X86 speculative load hardener
      --x86-winehstate                                                     - Insert stores for EH state numbers
      --x86argumentstackrebase                                             - Argument Stack Rebase
  --riscv-use-aa                                                        - Enable the use of AA during codegen.
  --runtime-counter-relocation                                          - Enable relocating counters at runtime.
  --safepoint-ir-verifier-print-only                                    - 
  --sample-profile-check-record-coverage=<N>                            - Emit a warning if less than N% of records in the input profile are matched to the IR.
  --sample-profile-check-sample-coverage=<N>                            - Emit a warning if less than N% of samples in the input profile are matched to the IR.
  --sample-profile-max-propagate-iterations=<uint>                      - Maximum number of iterations to go through when propagating sample block/edge weights through the CFG.
  --sampled-instr-burst-duration=<uint>                                 - Set the profile instrumentation burst duration, which can range from 1 to the value of 'sampled-instr-period' (0 is invalid). This number of samples will be recorded for each 'sampled-instr-period' count update. Setting to 1 enables simple sampling, in which case it is recommended to set 'sampled-instr-period' to a prime number.
  --sampled-instr-period=<uint>                                         - Set the profile instrumentation sample period. A sample period of 0 is invalid. For each sample period, a fixed number of consecutive samples will be recorded. The number is controlled by 'sampled-instr-burst-duration' flag. The default sample period of 65536 is optimized for generating efficient code that leverages unsigned short integer wrapping in overflow, but this is disabled under simple sampling (burst duration = 1).
  --sampled-instrumentation                                             - Do PGO instrumentation sampling
  --save-temp-labels                                                    - Don't discard temporary labels
  --separate-named-sections                                             - Use separate unique sections for named sections
  --skip-ret-exit-block                                                 - Suppress counter promotion if exit blocks contain ret.
  --speculative-counter-promotion-max-exiting=<uint>                    - The max number of exiting blocks of a loop to allow  speculative counter promotion
  --speculative-counter-promotion-to-loop                               - When the option is false, if the target block is in a loop, the promotion will be disallowed unless the promoted counter  update can be further/iteratively promoted into an acyclic  region.
  --split-machine-functions                                             - Split out cold basic blocks from machine functions based on profile information
  --stack-size-section                                                  - Emit a section containing stack size metadata
  --stack-symbol-ordering                                               - Order local stack symbols.
  --stackrealign                                                        - Force align the stack to the minimum alignment
  --strict-dwarf                                                        - use strict dwarf
  --strip-debug                                                         - Strip debugger symbol info from translation unit
  --strip-named-metadata                                                - Strip module-level named metadata
  --summary-file=<string>                                               - The summary file to use for function importing.
  --swift-async-fp=<value>                                              - Determine when the Swift async frame pointer should be set
    =auto                                                               -   Determine based on deployment target
    =always                                                             -   Always set the bit
    =never                                                              -   Never set the bit
  --tailcallopt                                                         - Turn fastcc calls into tail calls by (potentially) changing ABI.
  --target-abi=<string>                                                 - The name of the ABI to be targeted from the backend.
  --thin-link-bitcode-file=<filename>                                   - A file in which to write minimized bitcode for the thin link only
  --thinlto-bc                                                          - Write output as ThinLTO-ready bitcode
  --thinlto-split-lto-unit                                              - Enable splitting of a ThinLTO LTOUnit
  --thread-model=<value>                                                - Choose threading model
    =posix                                                              -   POSIX thread model
    =single                                                             -   Single thread model
  --time-trace                                                          - Record time trace
  --time-trace-file=<filename>                                          - Specify time trace file destination
  --tls-size=<uint>                                                     - Bit size of immediate TLS offsets
  --type-based-intrinsic-cost                                           - Calculate intrinsics cost based only on argument types
  --unique-basic-block-section-names                                    - Give unique names to every basic block section
  --unique-section-names                                                - Give unique names to every section
  --use-ctors                                                           - Use .ctors instead of .init_array.
  --vec-extabi                                                          - Enable the AIX Extended Altivec ABI.
  --verify-debuginfo-preserve                                           - Start the pipeline with collecting and end it with checking of debug info preservation.
  --verify-di-preserve-export=<filename>                                - Export debug info preservation failures into specified (JSON) file (should be abs path as we use append mode to insert new JSON objects)
  --verify-each                                                         - Verify after each transform
  --verify-each-debuginfo-preserve                                      - Start each pass with collecting and end it with checking of debug info preservation.
  --verify-legalizer-debug-locs=<value>                                 - Verify that debug locations are handled
    =none                                                               -   No verification
    =legalizations                                                      -   Verify legalizations
    =legalizations+artifactcombiners                                    -   Verify legalizations and artifact combines
  --verify-region-info                                                  - Verify region info (time consuming)
  --vp-counters-per-site=<number>                                       - The average number of profile counters allocated per value profiling site.
  --vp-static-alloc                                                     - Do static counter allocation for value profiler
  --wholeprogramdevirt-cutoff=<uint>                                    - Max number of devirtualizations for devirt module pass
  --write-experimental-debuginfo                                        - Write debug info in the new non-intrinsic format. Has no effect if --preserve-input-debuginfo-format=true.
  --x86-align-branch=<string>                                           - Specify types of branches to align (plus separated list of types):
                                                                          jcc      indicates conditional jumps
                                                                          fused    indicates fused conditional jumps
                                                                          jmp      indicates direct unconditional jumps
                                                                          call     indicates direct and indirect calls
                                                                          ret      indicates rets
                                                                          indirect indicates indirect unconditional jumps
  --x86-align-branch-boundary=<uint>                                    - Control how the assembler should align branches with NOP. If the boundary's size is not 0, it should be a power of 2 and no less than 32. Branches will be aligned to prevent from being across or against the boundary of specified size. The default value 0 does not align branches.
  --x86-branches-within-32B-boundaries                                  - Align selected instructions to mitigate negative performance impact of Intel's micro code update for errata skx102.  May break assumptions about labels corresponding to particular instructions, and should be used with caution.
  --x86-pad-max-prefix-size=<uint>                                      - Maximum number of prefixes to use for padding
  --x86-relax-relocations                                               - Emit GOTPCRELX/REX_GOTPCRELX/CODE_4_GOTPCRELX instead of GOTPCREL on x86-64 ELF
  --x86-sse2avx                                                         - Specify that the assembler should encode SSE instructions with VEX prefix
  --xcoff-traceback-table                                               - Emit the XCOFF traceback table
  --xray-function-index                                                 - Emit xray_fn_idx section

Generic Options:

  --help                                                                - Display available options (--help-hidden for more)
  --help-list                                                           - Display list of available options (--help-list-hidden for more)
  --version                                                             - Display the version of this program

```