# RISCV environment variable must be set

CC=$(RISCV)/bin/riscv64-unknown-elf-gcc
OBJCOPY=$(RISCV)/bin/riscv64-unknown-elf-objcopy
CFLAGS=-march=rv64imac -mcmodel=medany -O2 -std=gnu11 -Wall -nostartfiles 
CFLAGS+= -fno-common -g -DENTROPY=0 -mabi=lp64 -DNONSMP_HART=0 
CFLAGS+= -I $(BOOTROM_DIR)/include -I.
LFLAGS=-static -nostdlib -L $(BOOTROM_DIR)/linker -T sdboot.elf.lds

dts := $(BUILD_DIR)/$(CONFIG_PROJECT).$(CONFIG).dts
dtb := $(BUILD_DIR)/$(CONFIG_PROJECT).$(CONFIG).dtb
uboot_src := $(BOOTROM_DIR)/u-boot
uboot_dir := $(BUILD_DIR)/u-boot
uboot := $(uboot_dir)/u-boot

$(dtb): $(dts)
	dtc -I dts -O dtb -o $@ $<

.PHONY: dtb
dtb: $(dtb)

CROSS_COMPILE=$(RISCV)/bin/riscv64-unknown-linux-gnu-

elf := $(BUILD_DIR)/u-boot.elf
$(elf): $(dtb)
	rm -rf $(uboot_dir)
	mkdir -p $(uboot_dir)
	$(MAKE) -C $(uboot_src) O=$(uboot_dir) sifive_uX_rom_defconfig
	$(MAKE) -C $(uboot_src) O=$(uboot_dir) CROSS_COMPILE=$(CROSS_COMPILE)
	cp $(uboot_dir)/u-boot $(elf)

.PHONY: elf
elf: $(elf)

bin := $(BUILD_DIR)/u-boot.bin
$(bin): $(elf)
	$(OBJCOPY) -O binary $< $@

.PHONY: bin
bin: $(bin)

hex := $(BUILD_DIR)/u-boot.hex
$(hex): $(bin)
	od -t x4 -An -w4 -v $< > $@

.PHONY: hex
hex: $(hex)

romgen := $(BUILD_DIR)/rom.v
$(romgen): $(hex)
	$(rocketchip_dir)/scripts/vlsi_rom_gen $(ROMCONF) $< > $@

.PHONY: romgen
romgen: $(romgen)

.PHONY: clean
clean::
	rm -rf $(hex) $(elf)
