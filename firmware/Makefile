test: testbench.vvp start.hex
	vvp -N testbench.vvp

test_trace: testbench.vvp start.hex
	vvp -N $< +trace
	python3 ../showtrace.py testbench.trace start.rv32.elf > testbench.ins

test_nola: testbench_nola.vvp start.hex
	vvp -N testbench_nola.vvp

timing: timing.txt
	grep '^##' timing.txt | gawk 'x != "" {print x,$$3-y;} {x=$$2;y=$$3;}' | sort | uniq -c | \
		gawk '{printf("%03d-%-7s %2d %-8s (%d)\n",$$3,$$2,$$3,$$2,$$1);}' | sort | cut -c13-

timing.txt: timing.vvp start.hex
	vvp -N timing.vvp > timing.txt

testbench.vvp: testbench.v ../picorv32.v
	iverilog -o testbench.vvp testbench.v ../picorv32.v
	chmod -x testbench.vvp

testbench_nola.vvp: testbench_nola.v ../picorv32.v
	iverilog -o testbench_nola.vvp testbench_nola.v ../picorv32.v
	chmod -x testbench_nola.vvp

timing.vvp: testbench.v ../picorv32.v
	iverilog -o timing.vvp -DTIMING testbench.v ../picorv32.v
	chmod -x timing.vvp

start.hex: start.rv32.elf
	$(TOOLCHAIN_PREFIX)objcopy -O verilog $< $@

bootrom_img = bootrom.rv32.img bootrom.rv32.simple.img start.rv32.img
bootrom_dump = bootrom.rv32.dump bootrom.rv32.simple.dump start.rv32.dump

CROSS_COMPILE := riscv-none-embed-
# CROSS_COMPILE := riscv32-unknown-elf-
GCC=$(CROSS_COMPILE)gcc
CFLAGS_RV32=-mabi=ilp32 -march=rv32im
OBJCOPY=$(CROSS_COMPILE)objcopy
OBJDUMP=$(CROSS_COMPILE)objdump

START_FILES := irq.c print.c hello.c # multest.c sieve.c stats.c

default: img dump

img: $(bootrom_img)

dump: $(bootrom_dump)

%.img: %.elf
	$(OBJCOPY) -O binary --change-addresses=-0x10000 $< $@

start.rv32.elf: start.S linker.ld
	$(GCC) $(CFLAGS_RV32) -Tlinker.ld start.S $(START_FILES) -nostdlib -static -o start.rv32.elf

%.rv32.elf: %.S linker.ld
	$(GCC) $(CFLAGS_RV32) -Tlinker.ld $< -nostdlib -static -o $@

%.rv32.simple.elf: %_simple.S linker.ld
	$(GCC) $(CFLAGS_RV32) -Tlinker.ld bootrom_simple.S -nostdlib -static -o $@

%.dump: %.elf
	$(OBJDUMP) -d $< > $@

clean:
	rm -f *.elf *.dump *.img

