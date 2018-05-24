
# XXX
set(ISYS8010_SOURCES
  ../common/i8080.c
  ../common/i8251.c
  ../common/i8255.c
  ../common/ieprom.c
  ../common/iram8.c
  ../common/isbc064.c
  ../common/isbc201.c
  ../common/isbc202.c
  ../common/multibus.c
  ../common/zx200a.c
  isbc8010.c
  isys8010_sys.c
  )
add_simulator(isys8010)
