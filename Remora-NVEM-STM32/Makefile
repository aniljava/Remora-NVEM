# Compiler and Tools
CC := arm-none-eabi-gcc
CXX := arm-none-eabi-g++
AS := arm-none-eabi-as
OBJCOPY := arm-none-eabi-objcopy
OBJDUMP := arm-none-eabi-objdump
SIZE := arm-none-eabi-size

# MCU and Processor Flags


# Directories
SUBDIRS := Core Drivers LWIP Middlewares

# Find all source files
C_SOURCES := $(shell find $(SUBDIRS) -name '*.c')
CPP_SOURCES := $(shell find $(SUBDIRS) -name '*.cpp')
ASM_SOURCES := $(shell find $(SUBDIRS) -name '*.s')

# Object files
OBJS := $(sort $(C_SOURCES:.c=.o) $(CPP_SOURCES:.cpp=.o) $(ASM_SOURCES:.s=.o))

# Include Directories
INCLUDES := \
	-I./LWIP/App \
	-I./LWIP/Target \
	-I./Core/Inc \
	-I./Middlewares/Third_Party/LwIP/src/include \
	-I./Middlewares/Third_Party/LwIP/system \
	-I./Drivers/STM32F2xx_HAL_Driver/Inc \
	-I./Drivers/STM32F2xx_HAL_Driver/Inc/Legacy \
	-I./Middlewares/Third_Party/LwIP/src/include/netif/ppp \
	-I./Drivers/CMSIS/Device/ST/STM32F2xx/Include \
	-I./Middlewares/Third_Party/LwIP/src/include/lwip \
	-I./Middlewares/Third_Party/LwIP/src/include/lwip/apps \
	-I./Middlewares/Third_Party/LwIP/src/include/lwip/priv \
	-I./Middlewares/Third_Party/LwIP/src/include/lwip/prot \
	-I./Middlewares/Third_Party/LwIP/src/include/netif \
	-I./Middlewares/Third_Party/LwIP/src/include/posix \
	-I./Middlewares/Third_Party/LwIP/src/include/posix/sys \
	-I./Middlewares/Third_Party/LwIP/system/arch \
	-I./Drivers/CMSIS/Include

# Linker script
LDSCRIPT := STM32F207VCTX_FLASH.ld

# Compiler flags
CFLAGS := \
	-mcpu=cortex-m3 \
	-std=gnu11 \
	-DUSE_HAL_DRIVER \
	-DSTM32F207xx \
	-c \
	$(INCLUDES) \
	-Os \
	-ffunction-sections \
	-fdata-sections \
	-Wall \
	-fstack-usage \
	-MMD \
	-MP \
	-MF"$(@:%.o=%.d)" \
	-MT"$@" \
	--specs=nano.specs \
	-mfloat-abi=soft \
	-mthumb



CXXFLAGS := \
	-mcpu=cortex-m3 \
	-std=gnu++14 \
	-DUSE_HAL_DRIVER \
	-DSTM32F207xx \
	-c \
	$(INCLUDES) \
	-Os \
	-ffunction-sections \
	-fdata-sections \
	-fno-exceptions \
	-fno-rtti \
	-fno-use-cxa-atexit \
	-Wall \
	-fstack-usage \
	-MMD \
	-MP \
	-MF"$(@:%.o=%.d)" \
	-MT"$@" \
	--specs=nano.specs \
	-mfloat-abi=soft \
	-mthumb




LDFLAGS := -T$(LDSCRIPT) \
	-Wl,-Map=$(BUILD_DIR)/$(TARGET).map,--cref,--gc-sections \
	-static \
	-Wl,--start-group -lc -lm -lstdc++ -lsupc++ -Wl,--end-group

# Build artifact
BUILD_DIR := ./build
TARGET := Remora-NVEM

# Final binary and ELF
BIN := $(BUILD_DIR)/$(TARGET).bin
ELF := $(BUILD_DIR)/$(TARGET).elf
LIST := $(BUILD_DIR)/$(TARGET).list


# All Target
all: $(BUILD_DIR) $(ELF) $(BIN)

$(BUILD_DIR):
	mkdir -p $@

# Compile C files
%.o: %.c
	$(CC) $(CFLAGS) $< -o $@

# Compile C++ files
%.o: %.cpp
	$(CXX) $(CXXFLAGS) $< -o $@

# Assemble
%.o: %.s
	$(AS) $(MCU) -c $< -o $@

# Link
$(ELF): $(OBJS)
	arm-none-eabi-g++ \
	$(OBJS) \
	-mcpu=cortex-m3 \
	-TSTM32F207VCTX_FLASH.ld \
	--specs=nosys.specs \
	-Wl,-Map=$(BUILD_DIR)/$(TARGET).map \
	-Wl,--gc-sections \
	-static \
	--specs=nano.specs \
	-mfloat-abi=soft \
	-mthumb \
	-Wl,--start-group \
	-lc \
	-lm \
	-lstdc++ \
	-lsupc++ \
	-Wl,--end-group \
	-o \
	$@
	
	$(SIZE) $@

# Create binary
$(BIN): $(ELF)
	$(OBJCOPY) -O binary $< $@

$(LIST): $(ELF)
	$(OBJDUMP) -h -S $(ELF) > $@


flash:
	# picoprobe or cmsis-dap based programmer.
	openocd -f interface/cmsis-dap.cfg -f target/stm32f2x.cfg \
	-c "init" \
	-c "reset init" \
	-c "flash write_image erase $(BIN) 0x08000000" \
	-c "reset run" \
	-c "shutdown"

	# ST-Link (Uncomment the following line if you are using ST-Link)
	# st-flash write $(BIN) 0x08000000



# Clean
clean:
	rm -rf $(BUILD_DIR)
	find $(SUBDIRS) -type f \( -name "*.o" -o -name "*.d" -o -name "*.map" -o -name "*.lst" -o -name "*.su" \) -delete

# Debug target
debug:
	@echo "Object files:"
	@echo $(OBJS)

.PHONY: all clean debug