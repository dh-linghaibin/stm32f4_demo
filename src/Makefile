# STM32 Makefile Script (coppied from Gazzali Kernel Project)

CC          =   arm-none-eabi-gcc
LD          =   arm-none-eabi-gcc
GDB         =   arm-none-eabi-gdb
OBJCOPY     =   arm-none-eabi-objcopy
SIZE		=	arm-none-eabi-size

CODE    	=	code

SOURCES		=	arch/cm4/lib/core/startup_stm32f40_41xxx.s \
                arch/cm4/lib/core/system_stm32f4xx.c \
    	        arch/cm4/lib/src/stm32f4xx_rcc.c	\
                arch/cm4/lib/src/stm32f4xx_gpio.c \
                init/app.c #$(CODE)/startup_stm32f10x_hd.S $(CODE)/main.c

INCLUDES	=   -I. \
                -I arch/cm4/lib/inc \
                -I arch/cm4/lib/core

BUILDDIR	= 	output
OUTNAME		=	main


OBJECTS 	= 	$(addprefix $(BUILDDIR)/, $(addsuffix .o, $(basename $(SOURCES))))

ELF 		= 	$(BUILDDIR)/$(OUTNAME).elf
HEX 		= 	$(BUILDDIR)/$(OUTNAME).hex
BIN 		= 	$(BUILDDIR)/$(OUTNAME).bin

DEFS = -DUSE_STDPERIPH_DRIVER -DSTM32F40_41xxx #宏定义格式 -D+定义内容

CFLAGS  	= 	-g -Wall -I. -std=c99					\
				-nostdlib -fno-builtin -nodefaultlibs	\
				-mlittle-endian -mthumb -mcpu=cortex-m4 -mthumb-interwork \
				-nostartfiles -mfpu=fpv4-sp-d16 -mfloat-abi=hard -fpic  \
				$(DEFS) \
				$(addprefix -I,$(INCLUDES))


LDSCRIPT    :=  arch/cm4/stm32_flash.ld
LDFLAGS     :=  -Wl,-M=$(OUTNAME).map -g -T$(LDSCRIPT) $(CFLAGS)
#LDFLAGS     :=  -Wl,-M=$(OBJDIR)/$(OUTNAME).map -g -T$(LDSCRIPT) $(CFLAGS)

#Message Coloring Options

NO_COLOR			=	\033[0m
OK_COLOR			=	\033[32;01m
ERROR_COLOR			=	\033[31;01m
WARN_COLOR			=	\033[33;01m
MESSAGE_COLOR		=	\033[34;01m

COMPILING_MESSAGE	=	"$(OK_COLOR)[OK]$(MESSAGE_COLOR)CC $<$(NO_COLOR)"
CLEANING_MESSAGE	=	"$(OK_COLOR)[OK]$(MESSAGE_COLOR)Cleaning build files..."
CONVERTING_MESSAGE	=	"$(OK_COLOR)[OK]$(MESSAGE_COLOR)Converting to $@...$(NO_COLOR)"
LINKING_MESSAGE		=	"$(OK_COLOR)[OK]$(MESSAGE_COLOR)Linking...$(NO_COLOR)"
DEBUG_MESSAGE		=	"$(OK_COLOR)[OK]$(MESSAGE_COLOR)Starting Debugger$(NO_COLOR)"
SUCCESS_MESSAGE		=	"$(OK_COLOR)Compiled...\n\t$(ELF)\n\t$(BIN)\n\t$(HEX)$(NO_COLOR)"


all: $(BIN) $(HEX) size
	@echo 	 	$(SUCCESS_MESSAGE)

$(BIN): $(ELF)
	@$(OBJCOPY) -O 	binary $< $@
	@echo  		$(CONVERTING_MESSAGE)

$(HEX): $(ELF)
	@$(OBJCOPY) -O 	ihex $< $@
	@echo  		$(CONVERTING_MESSAGE)

$(ELF): $(OBJECTS)
	@$(LD) 		$(LDFLAGS) -o $@ $(OBJECTS) $(LDLIBS)
	@echo  		$(LINKING_MESSAGE)

$(BUILDDIR)/%.o: %.c
	@mkdir -p 	$(dir $@)
	@$(CC) 		$(CFLAGS) -c $< -o $@
	@echo  		$(COMPILING_MESSAGE)

$(BUILDDIR)/%.o: %.s
	@mkdir -p 	$(dir $@)
	@$(CC) 		$(CFLAGS) -c $< -o $@
	@echo  		$(COMPILING_MESSAGE)

size: $(ELF)
	@echo		"$(OK_COLOR)"
	@$(SIZE) 	$(ELF)
	@echo		"$(NO_COLOR)"
	
clean:
	@rm -rf 	$(BUILDDIR)
	@echo 		$(CLEANING_MESSAGE)


# Flash And Debug (OpenOCD)
debug-gdbserver: $(BIN)
	@$(OPENOCD) -f 	$(OPENOCD_CONF) &

debug-gdb: $(ELF)
	@echo 		$(DEBUG_MESSAGE)
	@$(GDB) 	$(ELF) -ex="target remote $(GDBHOST)"

debug-flash: $(BIN)
	st-flash write $(BIN) $(STARTADDRESS) 0x8000000
	#$(OPENOCD) -f 	$(OPENOCD_CONF) -c "program $(BIN) $(STARTADDRESS) verify"

