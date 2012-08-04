#¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤
#               USER 

MCU_TARGET      = atmega324p
DEFS            = -DF_CPU=8000000

PROJECTNAME=pingpong
PROJECT_BASE_DIR = ..

SRC=	pingpong.c 
        
INC= -I. \
	-I../inc

LIBS=

# Libraries that needs building:
include  $(PROJECT_BASE_DIR)/lib/rfm12/lib.mk
include  $(PROJECT_BASE_DIR)/lib/uart_lib/lib.mk
#include  $(PROJECT_BASE_DIR)/lib/xprintf/lib.mk

OPTIMIZE=s

LFUSE_VAL=E2
HFUSE_VAL=99
EFUSE_VAL=ff

AVRDUDE_PROGRAMMERID=jtag2isp
AVRDUDE_FLAGS=-B40
AVRDUDE_PORT=usb

# Set DEBUG to @ to remove extended 
DEBUG = @

#               END USER
#¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤

#¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤
#                FLAGS

HEXFORMAT=ihex

CFLAGS= $(INC) -g -mmcu=$(MCU_TARGET) $(DEFS) \
        -O$(OPTIMIZE) -fpack-struct \
        -fshort-enums -funsigned-bitfields \
        -funsigned-char -Wall -Wstrict-prototypes \
		-Wa,-ahlms=$(firstword                  \
		$(filter %.lst, $(<:.c=.lst)))

CPPFLAGS=-fno-exceptions               \
	-Wa,-ahlms=$(firstword         \
	$(filter %.lst, $(<:.cpp=.lst))\
	$(filter %.lst, $(<:.cc=.lst)) \
	$(filter %.lst, $(<:.C=.lst)))

ASMFLAGS =-I. $(INC) -mmcu=$(MCU_TARGET)        \
	-x assembler-with-cpp            \
	-Wa,-gstabs,-ahlms=$(firstword   \
		$(<:.S=.lst) $(<.s=.lst))


LDFLAGS=-Wl,-Map,$(TRG).map -mmcu=$(MCU_TARGET) \
	-lm $(LIBS)
	
SIZE_FLAGS=--mcu=$(MCU_TARGET) --format=avr
#              END FLAGS
#¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤

#¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤
#               BINARIES
CC=avr-gcc
OBJCOPY=avr-objcopy
OBJDUMP=avr-objdump
SIZE=avr-size
AVRDUDE=avrdude
REMOVE=rm -f
#              END BINARIES
#¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤

#¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤
#               FILES
TRG=$(PROJECTNAME).elf
DUMPTRG=$(PROJECTNAME).s

HEXROMTRG=$(PROJECTNAME).hex 
HEXTRG=$(HEXROMTRG) $(PROJECTNAME).ee.hex
GDBINITFILE=gdbinit-$(PROJECTNAME)

# Define all object files.
CPPFILES=$(filter %.cpp, $(SRC))
CCFILES=$(filter %.cc, $(SRC))
BIGCFILES=$(filter %.C, $(SRC))
CFILES=$(filter %.c, $(SRC))
ASMFILES=$(filter %.S, $(SRC))

OBJDEPS=$(CFILES:.c=.o)    \
	$(CPPFILES:.cpp=.o)\
	$(BIGCFILES:.C=.o) \
	$(CCFILES:.cc=.o)  \
	$(ASMFILES:.S=.o)

LST=$(filter %.lst, $(OBJDEPS:.o=.lst))

GENASMFILES=$(filter %.s, $(OBJDEPS:.o=.s)) 

DEPENDFILE = .depend

.SUFFIXES : .c .cc .cpp .C .o .out .s .S \
	.hex .ee.hex .h .hh .hpp


.PHONY: writeflash clean stats gdbinit stats
#              END FILES
#¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤

#¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤
#               TARGETS
# all, disasm, stats, size, hex, 
# flash/install, clean, fuse

all: dep $(TRG) hex size

disasm: $(DUMPTRG) stats

stats: $(TRG)
	$(OBJDUMP) -h $(TRG)
	$(SIZE) $(TRG) 
	
size: $(TRG)
	@echo "\n- $(TRG) -"
	$(DEBUG)$(SIZE) $(SIZE_FLAGS) $(TRG) 
        
hex: $(HEXROMTRG)

fuse:
	$(AVRDUDE) -c $(AVRDUDE_PROGRAMMERID) $(AVRDUDE_FLAGS)  \
	-p $(MCU_TARGET) -P $(AVRDUDE_PORT) -u                  \
	-Ulfuse:w:0x$(LFUSE_VAL):m                              \
	-Uhfuse:w:0x$(HFUSE_VAL):m                              \
	-Uefuse:w:0x$(EFUSE_VAL):m


flash: hex
	$(AVRDUDE) -c $(AVRDUDE_PROGRAMMERID) $(AVRDUDE_FLAGS)  \
	 -p $(MCU_TARGET) -P $(AVRDUDE_PORT) -e        \
	-U flash:w:$(HEXROMTRG)	

install: flash

$(DUMPTRG): $(TRG) 
	$(OBJDUMP) -S  $< > $@


$(TRG): $(OBJDEPS) 
	@echo ">> Building: $(TRG) <<"
	$(DEBUG)$(CC) $(LDFLAGS) -o $(TRG) $(OBJDEPS)

%.s: %.c
	@echo ">> Building: $@ <<"
	$(DEBUG)$(CC) -S $(CFLAGS) $< -o $@

%.s: %.S
	@echo ">> Building: $@ <<"
	$(DEBUG)$(CC) -S $(ASMFLAGS) $< > $@

.cpp.s .cc.s .C.s :
	@echo ">> Building: $@ <<"
	$(DEBUG)$(CC) -S $(CFLAGS) $(CPPFLAGS) $< -o $@

.c.o: 
	@echo ">> Building: $@ <<"
	$(DEBUG)$(CC) $(CFLAGS) -c $< -o $@

.cc.o .cpp.o .C.o :
	@echo ">> Building: $@ <<"
	$(DEBUG)$(CC) $(CFLAGS) $(CPPFLAGS) -c $< -o $@

.S.o :
	@echo ">> Building: $@ <<"
	$(DEBUG)$(CC) $(ASMFLAGS) -c $< -o $@

%.hex: %.elf
	@echo ">> Building: $@ <<"
	$(DEBUG)$(OBJCOPY) -j .text                    \
		-j .data                       \
		-O $(HEXFORMAT) $< $@

dep: $(SRC)
	@echo ">> Checking dependencies <<"
	$(DEBUG)$(CC) $(INC) -mmcu=$(MCU_TARGET) $(DEFS) -MM $(SRC) > $(DEPENDFILE)

-include $(DEPENDFILE)

clean:
	@echo ">> Clean-up <<"
	$(DEBUG)$(REMOVE) $(TRG) $(TRG).map $(DUMPTRG)
	$(DEBUG)$(REMOVE) $(OBJDEPS)
	$(DEBUG)$(REMOVE) $(LST) $(GDBINITFILE)
	$(DEBUG)$(REMOVE) $(GENASMFILES)
	$(DEBUG)$(REMOVE) $(HEXTRG)
	$(DEBUG)$(REMOVE) $(DEPENDFILE)

#            END TARGETS
#¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤
	
# DO NOT DELETE
