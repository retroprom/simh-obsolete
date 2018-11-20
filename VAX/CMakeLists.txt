
# DEC VAX common bits
set(VAX_DEFINITIONS
  VM_VAX
  USE_INT64
  USE_ADDR64
  )
set(VAX_INCLUDES
  ../PDP11
  )
set(VAX_SOURCES
  vax_cis.c
  vax_cmode.c
  vax_cpu1.c
  vax_cpu.c
  vax_fpa.c
  vax_mmu.c
  vax_octa.c
  vax_sys.c
  vax_syscm.c
  vax_watch.c
  ../PDP11/pdp11_rl.c
  ../PDP11/pdp11_rq.c
  ../PDP11/pdp11_ts.c
  ../PDP11/pdp11_dz.c
  ../PDP11/pdp11_lp.c
  ../PDP11/pdp11_tq.c
  ../PDP11/pdp11_td.c
  ../PDP11/pdp11_cr.c
  ../PDP11/pdp11_vh.c
  ../PDP11/pdp11_io_lib.c
  )
function(add_vax_simulator NAME)
  string(TOUPPER ${NAME} NAMEU)
  add_simulator(${NAME})
  add_simulator_network(${NAME})
  if(BUILD_ALL OR SIMULATOR_${NAMEU}_ENABLE)
    target_include_directories(${NAME} PUBLIC ${VAX_INCLUDES})
  endif()
endfunction()

# DEC Massbus common bits
set(VAX_MASSBUS_SOURCES
  vax7x0_mba.c
  ../PDP11/pdp11_rp.c
  ../PDP11/pdp11_tu.c
  )

# DEC Q-bus common bits
set(VAX_QBUS_SOURCES
  ../PDP11/pdp11_xq.c
  )

# DEC Unibus common bits
set(VAX_UNIBUS_SOURCES
  ../PDP11/pdp11_xu.c
  ../PDP11/pdp11_ry.c
  ../PDP11/pdp11_hk.c
  ../PDP11/pdp11_dmc.c
  ../PDP11/pdp11_tc.c
  ../PDP11/pdp11_rk.c
  )

# DEC VAX6XX Q-bus common bits
set(VAX6XX_SOURCES
  ${VAX_SOURCES}
  ${VAX_QBUS_SOURCES}
  vax_2681.c
  vax_lk.c
  vax_vc.c
  vax_vs.c
  )

# DEC VAX610 - MicroVAX I
set(VAX610_DEFINITIONS
  ${VAX_DEFINITIONS}
  VAX_610
  )
set(VAX610_SOURCES
  ${VAX6XX_SOURCES}
  vax610_io.c
  vax610_mem.c
  vax610_stddev.c
  vax610_sysdev.c
  vax610_syslist.c
  )
add_vax_simulator(vax610)
add_simulator_graphics(vax610)

# DEC VAX630 - MicroVAX II
set(VAX630_DEFINITIONS
  ${VAX_DEFINITIONS}
  VAX_630
  )
set(VAX630_SOURCES
  ${VAX6XX_SOURCES}
  vax630_io.c
  vax630_stddev.c
  vax630_sysdev.c
  vax630_syslist.c
  )
add_vax_simulator(vax630)
add_simulator_graphics(vax630)

# DEC VAX620 - rtVAX 1000
set(VAX620_DEFINITIONS
  ${VAX_DEFINITIONS}
  VAX_620
  )
set(VAX620_SOURCES
  ${VAX630_SOURCES}
  )
add_vax_simulator(vax620)

# DEC VAX650 - MicroVAX III
set(VAX650_DEFINITIONS
  ${VAX_DEFINITIONS}
  VAX_650
  )
set(VAX650_SOURCES
  ${VAX6XX_SOURCES}
  vax_io.c
  vax_stddev.c
  vax_sysdev.c
  vax_syslist.c
  )
add_vax_simulator(vax650)
add_simulator_graphics(vax650)

# DEC VAX7XX - Unibus VAXen
set(VAX7XX_SOURCES
  ${VAX_SOURCES}
  ${VAX_UNIBUS_SOURCES}
  )

# DEC VAX730 - VAX 11/730
set(VAX730_DEFINITIONS
  ${VAX_DEFINITIONS}
  VAX_730
  )
set(VAX730_SOURCES
  ${VAX7XX_SOURCES}
  vax730_mem.c
  vax730_rb.c
  vax730_stddev.c
  vax730_sys.c
  vax730_syslist.c
  vax730_uba.c
  )
add_vax_simulator(vax730)

# DEC VAX750 - VAX 11/750
set(VAX750_DEFINITIONS
  ${VAX_DEFINITIONS}
  VAX_750
  )
set(VAX750_SOURCES
  ${VAX7XX_SOURCES}
  ${VAX_MASSBUS_SOURCES}
  vax750_cmi.c
  vax750_mem.c
  vax750_stddev.c
  vax750_syslist.c
  vax750_uba.c
  )
add_vax_simulator(vax750)

# DEC VAX780 - VAX 11/780
set(VAX780_DEFINITIONS
  ${VAX_DEFINITIONS}
  VAX_780
  )
set(VAX780_SOURCES
  ${VAX7XX_SOURCES}
  ${VAX_MASSBUS_SOURCES}
  vax780_fload.c
  vax780_mem.c
  vax780_sbi.c
  vax780_stddev.c
  vax780_syslist.c
  vax780_uba.c
  )
add_vax_simulator(vax780)

# DEC VAX860 - VAX 8600
set(VAX860_DEFINITIONS
  ${VAX_DEFINITIONS}
  VAX_860
  )
set(VAX860_SOURCES
  ${VAX7XX_SOURCES}
  ${VAX_MASSBUS_SOURCES}
  vax780_uba.c
  vax860_abus.c
  vax860_sbia.c
  vax860_stddev.c
  vax860_syslist.c
  )
add_vax_simulator(vax860)
