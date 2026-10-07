# CVA5 formal verification

JasperGold flows for unit-level AXI master proofs and full-core reachability.

Run from `formal/`:

```sh
make formal-filelist
make formal-elab GUI=0 ENGINE_MODE=auto

make formal-axi-master-proof-framework GUI=1 ENGINE_MODE=auto
make formal-axi-master-read-closure GUI=0 ENGINE_MODE=auto
make formal-axi-master-write-closure GUI=0 ENGINE_MODE=auto
make formal-axi-master-combined-closure GUI=0 ENGINE_MODE=auto

make formal-cva5-axi-proof-framework GUI=1 TIME_LIMIT=5m ENGINE_MODE=auto
make formal-cva5-axi-load-list-stages
make formal-cva5-axi-load-stage STAGE=instruction_return GUI=0 TIME_LIMIT=5m ENGINE_MODE=auto
make formal-cva5-axi-load-reachability GUI=1 TIME_LIMIT=5m ENGINE_MODE=auto
make formal-cva5-reachability-stage STAGE=instruction_fetch GUI=0 TIME_LIMIT=2m ENGINE_MODE=auto
make formal-cva5-reachability-stage STAGE=axi_ar_wait_1 GUI=0 TIME_LIMIT=2m
```

## Targets

- `formal-filelist`, `formal-elab`: generate the filelist; analyze and elaborate the full-core wrapper.
- `formal-axi-master-proof-framework`: unit-level Jasper proof structure and checker coverage.
- `formal-axi-master-read-closure`, `formal-axi-master-write-closure`, `formal-axi-master-combined-closure`: unit smoke, safety, and checker proofs.
- `formal-cva5-axi-proof-framework`: full-core assumptions, reachability, AXI safety, and lifecycle tasks.
- `formal-cva5-axi-load-stage STAGE=<name>`: run one anchored LW stage.
- `formal-cva5-axi-load-reachability`: run anchored LW stages in order.
- `formal-cva5-reachability-stage STAGE=<name>`: run one full-core reachability cover, including individual AXI timing profiles such as `axi_ar_wait_1`.
- `formal-list`, `formal-clean`: list Tcl targets; remove generated `runs/`.

## Proof boundary

- Unit harnesses prove read-only, write-only, and combined `axi_master` behavior.
- Full-core proofs use the embedded AXI interface and an anchored LW path.
- Full-core reachability covers the load path; it is not a whole-core coverage metric.
- AXI checker limits assume one outstanding transaction and legal response ordering.
- Ready bounds apply only to covers; safety proofs leave ready unbounded.
- Undriven AXI field checks are opt-in with `INCLUDE_UNDRIVEN_FIELDS=1`.

## Options

- `GUI=0`: batch mode.
- `ENGINE_MODE=auto`: default engine mode for unit and full-core flows.
- `TIME_LIMIT=5m`: focused proof limit; override for staged runs.
- `FRAMEWORK_RUN_PROOFS=0`, `CVA5_FRAMEWORK_RUN_PROOFS=0`: build framework tasks without running proof roots.
- `CVA5_FRAMEWORK_RUN_REACHABILITY=1`: enable broad reset, startup, and fetch covers; focused stages are the normal flow.
- `CVA5_FRAMEWORK_RUN_DEEP_RESPONSE=1`: enable resource-intensive LSU completion and data assertions.
- `CVA5_LOAD_MAX_JOBS=2`, `CVA5_LOAD_ORCHESTRATION=off`: focused load scheduling; load reachability chains covers.
- `CVA5_SAFETY_MAX_JOBS=2`, `CVA5_SAFETY_ORCHESTRATION=off`: focused safety scheduling; safety mode proves a selected assertion. These controls are separate because the execution models differ.
- `CVA5_LOAD_DUMP_TRACE=0`: set to `1` to export the reached load trace as VCD.
- `CVA5_USE_PROVEN_LW_LEMMAS=1`: enables proven-helper reuse for focused `LW_WRITEBACK_GUARANTEES` stages; set to `0` to run those selected guarantee proofs without the imported helper cuts.
- `ENGINE_MODE` is independent of load and safety scheduling.

## Layout

- `interfaces/`: reusable formal properties.
- `models/full_core/`: CVA5 wrappers, FBM, and fetch-local harness.
- `models/unit_axi_master/`: read, write, combined, and proof-framework harnesses.
- `models/legacy/`: legacy AXI master and checker harnesses.
- `scripts/tcl/`: Jasper targets.
- `scripts/run_jg.sh`: run-directory setup and Jasper launcher.
- `filelists/`, `runs/`: generated filelists and run data.

JasperGold (`jg`) must be on `PATH`. Site-specific environment setup may go in
`set_env.sh`; it is not part of the default flow.
