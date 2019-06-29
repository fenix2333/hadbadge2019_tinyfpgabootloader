PROJ=tinyfpgabl
CONSTR=had19_proto1.lpf
TRELLIS=/usr/share/trellis
SRC=bootloader.v pll_8_48.v ../../common/*.v 

#qspi, dual-spi, fast-read
FLASH_MODE=qspi
#2.4, 4.8, 9.7, 19.4, 38.8, 62.0
FLASH_FREQ=38.8


all: $(PROJ).svf multiboot.bin

$(PROJ).json: $(SRC)
	yosys -p "synth_ecp5  -top hadbadgebl -json $(PROJ).json" $(SRC)

%_out.config: %.json
	nextpnr-ecp5 --json $< --lpf $(CONSTR) --textcfg $@ --um-45k --package CABGA381 --speed 8

%.bit: %_out.config
	ecppack --spimode $(FLASH_MODE) --freq $(FLASH_FREQ) --svf-rowsize 100000 --svf $(PROJ).svf --input $< --bit $@

multiboot.bin: $(PROJ).bit soc.bit
	ecpmulti --flashsize 128 --input $(PROJ).bit --input soc.bit --address 0x180000 $@

$(PROJ).svf: $(PROJ).bit

prog: $(PROJ).svf
	openocd -f ../../../openocd.cfg -c "init; svf  $<; exit"

flash: $(PROJ).svf multiboot.bin
	make prog
	sleep 1
	tinyprog -p multiboot.bin -a 0

clean:
	rm -f $(PROJ).json $(PROJ).svf $(PROJ).bit $(PROJ)_out.config

.PHONY: prog clean
.PRECIOUS: $(PROJ).json $(PROJ)_out.config
