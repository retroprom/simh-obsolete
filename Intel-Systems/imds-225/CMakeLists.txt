
# XXX
set(IMDS225_SOURCES
  ../common/i8080.c
  ../common/i8251.c
  ../common/i8253.c
  ../common/i8255.c
  ../common/i8259.c
  ../common/ieprom.c
  ../common/ioc-cont.c
  ../common/multibus.c
  ../common/ipc.c
  ../common/ipc-cont.c
  ../common/iram8.c
  ../common/isbc201.c
  ../common/isbc202.c
  ../common/zx200a.c
  imds-225_sys.c
  )
add_simulator(imds225)
