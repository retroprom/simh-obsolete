
# XXX
set(ISYS8020_SOURCES
  ../common/i8080.c
  ../common/i8251.c
  ../common/i8255.c
  ../common/ieprom.c
  ../common/iram8.c
  ../common/isbc064.c
  ../common/i8259.c
  ../common/isbc201.c
  ../common/isbc202.c
  ../common/multibus.c
  ../common/zx200a.c
  isbc8020.c
  isys8020_sys.c
  )
add_simulator(isys8020)
