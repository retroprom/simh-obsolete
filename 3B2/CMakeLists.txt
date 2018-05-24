
# XXX
set(3B2_DEFINITIONS
  USE_ADDR64
  USE_INT64
  )
set(3B2_SOURCES
  3b2_cpu.c
  3b2_ctc.c
  3b2_dmac.c
  3b2_id.c
  3b2_if.c
  3b2_io.c
  3b2_iu.c
  3b2_mmu.c
  3b2_ports.c
  3b2_sys.c
  3b2_sysdev.c
  )
add_simulator(3b2)
