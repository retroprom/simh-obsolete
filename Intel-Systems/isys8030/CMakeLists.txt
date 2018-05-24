
# XXX
set(ISYS8030_SOURCES
  ../common/i8080.c
  ../common/i8251.c
  ../common/i8253.c
  ../common/i8255.c
  ../common/i8259.c
  ../common/ieprom.c
  ../common/iram8.c
  ../common/multibus.c
  ../common/isbc064.c
  ../common/isbc201.c
  ../common/isbc202.c
  ../common/zx200a.c
  isbc8030.c
  isys8030_sys.c
)
add_simulator(isys8030)
