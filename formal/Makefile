GUI ?= 1
TIME_LIMIT ?= 5m
USE_PROVEN_LEMMAS ?= 0
ENGINE_MODE ?= auto
INCLUDE_UNDRIVEN_FIELDS ?= 0
FRAMEWORK_RUN_PROOFS ?= 1
CVA5_FRAMEWORK_RUN_PROOFS ?= 1
CVA5_FRAMEWORK_RUN_REACHABILITY ?= 0
CVA5_FRAMEWORK_RUN_DEEP_RESPONSE ?= 0
CVA5_LOAD_MAX_JOBS ?= 2
CVA5_LOAD_ORCHESTRATION ?= off
CVA5_LOAD_DUMP_TRACE ?= 0
CVA5_SAFETY_MAX_JOBS ?= 2
CVA5_SAFETY_ORCHESTRATION ?= off
CVA5_USE_PROVEN_LW_LEMMAS ?= 1

CVA5_AXI_LOAD_STAGES = \
	reset_release \
	startup_complete \
	instruction_request \
	instruction_return \
	debug_fetch_request_alloc \
	debug_icache_line_complete \
	debug_icache_request_queued \
	debug_icache_restart \
	debug_icache_lw_hit \
	debug_fetch_lw_raw \
	fetch_complete \
	debug_lui_decode \
	debug_lui_issue \
	debug_lui_writeback \
	decode \
	debug_lw_source_mapping \
	issue \
	debug_lw_source_value \
	debug_lw_source_mismatch \
	lsu_command \
	debug_dtlb_bus_match \
	debug_lsq_load_valid \
	debug_axi_subunit_ready \
	lsu_request_accept \
	axi_master_requesting_read \
	axi_arvalid \
	axi_ar_handshake \
	axi_r_response \
	lsu_completion \
	full_lifecycle \
	writeback \
	retirement \
	x2_update

CVA5_AXI_SAFETY_STAGES = \
	load_accept_enters_requesting_read \
	requesting_read_drives_arvalid \
	embedded_arvalid_hold \
	embedded_araddr_stability \
	lw_decode_destination \
	lw_tracker_created \
	lw_tracker_holds \
	lw_id_to_phys_mapping \
	lw_tracker_clears \
	lw_rdata_capture \
	lw_rdata_holds \
	lw_retirement_tracker_clears \
	lw_rdata_matches_lsu_data \
	lw_writeback_destination \
	lw_writeback_data \
	lw_register_file_write_port \
	lw_register_file_update \
	lw_architectural_x2_update \
	lw_instruction_result

RUN_OPTS = GUI=$(GUI) TIME_LIMIT=$(TIME_LIMIT) ENGINE_MODE=$(ENGINE_MODE)
UNIT_OPTS = $(RUN_OPTS) INCLUDE_UNDRIVEN_FIELDS=0
RUN_JG = JG_TIME_LIMIT='$(TIME_LIMIT)' JG_ENGINE_MODE='$(ENGINE_MODE)' scripts/run_jg.sh
WRITE_FIELD_OPTS = AXI_WRITE_INCLUDE_UNDRIVEN_FIELDS='$(INCLUDE_UNDRIVEN_FIELDS)'
CVA5_FOCUSED_MODE_OPTS = JG_CVA5_FRAMEWORK_RUN_PROOFS=1 JG_CVA5_FRAMEWORK_RUN_REACHABILITY=0 JG_CVA5_FRAMEWORK_RUN_DEEP_RESPONSE=0

.DEFAULT_GOAL := help
.PHONY: formal-axi-property
.PHONY: formal-cva5-reachability-stage formal-cva5-axi-proof-framework formal-cva5-axi-load-stage formal-cva5-axi-load-reachability \
        formal-cva5-axi-load-list-stages formal-cva5-axi-safety-stage formal-cva5-axi-safety-list-stages formal-cva5-fetch-local-stage \
        formal-axi-master-read-property formal-axi-master-read-closure formal-axi-master-write-property formal-axi-master-write-closure \
        formal-axi-master-combined-property formal-axi-master-combined-all-safety formal-axi-master-combined-closure formal-axi-master-unit-closure \
        formal-axi-master-proof-framework formal-axi-checker-property

.PHONY: help formal-help formal-elab formal-filelist formal-clean formal-list
help formal-help:
	@echo "CVA5 formal targets"
	@echo "Run defaults: GUI=1 TIME_LIMIT=5m ENGINE_MODE=auto; use GUI=0 for batch mode."
	@echo
	@echo "Main unit-level closure commands:"
	@echo "  make formal-axi-master-proof-framework"
	@echo "    includes branch/statement COI Checker Coverage after the proof roots"
	@echo "    add FRAMEWORK_RUN_PROOFS=0 to inspect the unrun task/Proof Structure manifest"
	@echo "  make formal-axi-master-unit-closure"
	@echo "  make formal-axi-master-read-closure"
	@echo "  make formal-axi-master-write-closure"
	@echo "  make formal-axi-master-combined-closure"
	@echo
	@echo "Single-property debug commands:"
	@echo "  make formal-axi-master-read-property PROPERTY=<name>"
	@echo "  make formal-axi-master-write-property PROPERTY=<name>"
	@echo "  make formal-axi-master-combined-property PROPERTY=<name>"
	@echo "  make formal-axi-checker-property PROPERTY=<helper_name>"
	@echo
	@echo "Full-core and fetch reachability:"
	@echo "  make formal-cva5-axi-proof-framework"
	@echo "    runs embedded-instance safety roots and scoped COI coverage by default"
	@echo "    add CVA5_FRAMEWORK_RUN_REACHABILITY=1 to run broad base covers"
	@echo "    add CVA5_FRAMEWORK_RUN_DEEP_RESPONSE=1 for resource-intensive LSU/data checks"
	@echo "    add CVA5_FRAMEWORK_RUN_PROOFS=0 for task/Proof Structure setup only"
	@echo "  make formal-cva5-axi-load-reachability"
	@echo "  make formal-cva5-axi-load-stage STAGE=<stage>"
	@echo "  make formal-cva5-axi-load-list-stages"
	@echo "  make formal-cva5-axi-safety-stage STAGE=<stage>"
	@echo "  make formal-cva5-axi-safety-list-stages"
	@echo "  make formal-cva5-reachability-stage STAGE=<stage>"
	@echo "    timing profile: make formal-cva5-reachability-stage STAGE=axi_ar_wait_1"
	@echo "  make formal-cva5-fetch-local-stage STAGE=<stage>"
	@echo
	@echo "Utilities:"
	@echo "  make formal-elab"
	@echo "  make formal-filelist"
	@echo "  make formal-list"
	@echo
	@echo "INCLUDE_UNDRIVEN_FIELDS=1 enables optional checks for undriven AXI fields"
	@echo "at the direct axi_master boundary. Keep it at 0 for normal unit closure."

# ----------------------------------------------------------------------
# EMBEDDED AXI FOCUSED DEBUG
# ----------------------------------------------------------------------
formal-axi-property:
	@test -n "$(PROPERTY)" || (echo "error: set PROPERTY=<property label>" >&2; exit 1)
	@case '$(PROPERTY)' in helper_*) echo "error: use formal-axi-checker-property for helper_* properties" >&2; exit 1;; esac
	JG_PROPERTY='$(PROPERTY)' $(RUN_JG) --target axi_property --gui $(GUI)

# ----------------------------------------------------------------------
# FULL-CORE CVA5 WORKFLOWS
# ----------------------------------------------------------------------
formal-cva5-reachability-stage:
	@test -n "$(STAGE)" || (echo "error: set STAGE=<stage>" >&2; exit 1)
	JG_STAGE='$(STAGE)' $(RUN_JG) --target cva5_reachability_stage --gui $(GUI)

formal-cva5-axi-proof-framework:
	JG_CVA5_FRAMEWORK_RUN_PROOFS='$(CVA5_FRAMEWORK_RUN_PROOFS)' JG_CVA5_FRAMEWORK_RUN_REACHABILITY='$(CVA5_FRAMEWORK_RUN_REACHABILITY)' JG_CVA5_FRAMEWORK_RUN_DEEP_RESPONSE='$(CVA5_FRAMEWORK_RUN_DEEP_RESPONSE)' $(RUN_JG) --target cva5_axi_proof_framework --gui $(GUI)

formal-cva5-axi-load-stage:
	@test -n "$(STAGE)" || (echo "error: set STAGE=<stage>; run make formal-cva5-axi-load-list-stages" >&2; exit 1)
	JG_CVA5_FRAMEWORK_LOAD_STAGE='$(STAGE)' JG_CVA5_LOAD_MAX_JOBS='$(CVA5_LOAD_MAX_JOBS)' JG_CVA5_LOAD_ORCHESTRATION='$(CVA5_LOAD_ORCHESTRATION)' JG_CVA5_LOAD_DUMP_TRACE='$(CVA5_LOAD_DUMP_TRACE)' $(CVA5_FOCUSED_MODE_OPTS) $(RUN_JG) --target cva5_axi_proof_framework --gui $(GUI)

formal-cva5-axi-load-reachability:
	$(MAKE) formal-cva5-axi-load-stage STAGE=all $(RUN_OPTS)

formal-cva5-axi-load-list-stages:
	@echo "Anchored full-core LW reachability STAGE values:"
	@printf '  %s\n' $(CVA5_AXI_LOAD_STAGES)

formal-cva5-axi-safety-stage:
	@test -n "$(STAGE)" || (echo "error: set STAGE=<stage>; run make formal-cva5-axi-safety-list-stages" >&2; exit 1)
	JG_CVA5_FRAMEWORK_SAFETY_STAGE='$(STAGE)' JG_CVA5_SAFETY_MAX_JOBS='$(CVA5_SAFETY_MAX_JOBS)' JG_CVA5_SAFETY_ORCHESTRATION='$(CVA5_SAFETY_ORCHESTRATION)' JG_CVA5_USE_PROVEN_LW_LEMMAS='$(CVA5_USE_PROVEN_LW_LEMMAS)' $(CVA5_FOCUSED_MODE_OPTS) $(RUN_JG) --target cva5_axi_proof_framework --gui $(GUI)

formal-cva5-axi-safety-list-stages:
	@echo "Embedded full-core AXI safety STAGE values:"
	@printf '  %s\n' $(CVA5_AXI_SAFETY_STAGES)

formal-cva5-fetch-local-stage:
	@test -n "$(STAGE)" || (echo "error: set STAGE=<stage>; fetch-local examples: request_alloc, attr_valid_and_miss_data, fetch_complete_lui" >&2; exit 1)
	JG_STAGE='$(STAGE)' $(RUN_JG) --target cva5_fetch_local_stage --gui $(GUI)

# ----------------------------------------------------------------------
# UNIT AXI FOCUSED PROPERTIES
# ----------------------------------------------------------------------
formal-axi-master-read-property:
	@test -n "$(PROPERTY)" || (echo "error: set PROPERTY=<property label>" >&2; exit 1)
	JG_PROPERTY='$(PROPERTY)' AXI_READ_USE_PROVEN_LEMMAS='$(USE_PROVEN_LEMMAS)' $(RUN_JG) --target axi_master_read_property --gui $(GUI) --no-filelist

formal-axi-master-write-property:
	@test -n "$(PROPERTY)" || (echo "error: set PROPERTY=<property label>" >&2; exit 1)
	$(WRITE_FIELD_OPTS) JG_PROPERTY='$(PROPERTY)' $(RUN_JG) --target axi_master_write_property --gui $(GUI) --no-filelist

formal-axi-master-combined-property:
	@test -n "$(PROPERTY)" || (echo "error: set PROPERTY=<property label>" >&2; exit 1)
	$(WRITE_FIELD_OPTS) JG_PROPERTY='$(PROPERTY)' $(RUN_JG) --target axi_master_combined_property --gui $(GUI) --no-filelist

formal-axi-checker-property:
	@test -n "$(PROPERTY)" || (echo "error: set PROPERTY=<helper property label>" >&2; exit 1)
	JG_PROPERTY='$(PROPERTY)' $(RUN_JG) --target axi_checker_property --gui $(GUI) --no-filelist

# ----------------------------------------------------------------------
# UNIT AXI CLOSURE AND FRAMEWORK
# ----------------------------------------------------------------------
formal-axi-master-read-closure:
	$(RUN_JG) --target axi_master_read_smoke --gui $(GUI) --no-filelist
	$(RUN_JG) --target axi_master_read_lifecycle_safety --gui $(GUI) --no-filelist
	$(RUN_JG) --target axi_master_read_checker --gui $(GUI) --no-filelist

formal-axi-master-write-closure:
	AXI_WRITE_INCLUDE_UNDRIVEN_FIELDS='0' $(RUN_JG) --target axi_master_write_smoke --gui $(GUI) --no-filelist
	AXI_WRITE_INCLUDE_UNDRIVEN_FIELDS='0' $(RUN_JG) --target axi_master_write_lifecycle_safety --gui $(GUI) --no-filelist
	AXI_WRITE_INCLUDE_UNDRIVEN_FIELDS='0' $(RUN_JG) --target axi_master_write_checker --gui $(GUI) --no-filelist

formal-axi-master-combined-all-safety:
	$(WRITE_FIELD_OPTS) $(RUN_JG) --target axi_master_combined_all_safety --gui $(GUI) --no-filelist

formal-axi-master-combined-closure:
	AXI_WRITE_INCLUDE_UNDRIVEN_FIELDS='0' $(RUN_JG) --target axi_master_combined_smoke --gui $(GUI) --no-filelist
	AXI_WRITE_INCLUDE_UNDRIVEN_FIELDS='0' $(RUN_JG) --target axi_master_combined_cross_safety --gui $(GUI) --no-filelist
	AXI_WRITE_INCLUDE_UNDRIVEN_FIELDS='0' $(RUN_JG) --target axi_master_combined_checker --gui $(GUI) --no-filelist

formal-axi-master-unit-closure:
	$(MAKE) formal-axi-master-read-closure $(RUN_OPTS)
	$(MAKE) formal-axi-master-write-closure $(UNIT_OPTS)
	$(MAKE) formal-axi-master-combined-closure $(UNIT_OPTS)

formal-axi-master-proof-framework:
	$(WRITE_FIELD_OPTS) JG_FRAMEWORK_RUN_PROOFS='$(FRAMEWORK_RUN_PROOFS)' $(RUN_JG) --target axi_master_proof_framework --gui $(GUI) --no-filelist

# ----------------------------------------------------------------------
# FORMAL UTILITIES
# ----------------------------------------------------------------------
formal-elab:
	$(RUN_JG) --target formal_elab --gui $(GUI)

formal-filelist:
	bash scripts/populate_cva5_rtl_vfile.sh

formal-clean:
	rm -rf runs

formal-list:
	@find scripts/tcl -maxdepth 1 -name '*.tcl' ! -name '_*' -printf '%f\n' | sed 's/\.tcl$$//'
