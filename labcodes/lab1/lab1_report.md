#lab1实验报告
##练习1：理解通过make生成执行文件的过程
###1.1操作系统镜像文件ucore.img是如何一步一步生成的？(需要比较详细地解释Makefile中每一条相关命令和命令参数的含义，以及说明命令导致的结果)
	ucore.img
	| $(UCOREIMG): $(kernel) $(bootblock)
	| 
	| > $(kernel)
	|   | $(kernel): tools/kernel.ld
	|   | $(kernel): $(KOBJS)
	|   |
	|   | > $(KOBJS)
	|   |   | $(call add_files_cc,$(call listf_cc,$(KSRCDIR)),kernel,$(KCFLAGS))
	|   |   |   | 这里生成.o文件，例如生成init.o的实际命令是gcc -Ikern/init/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/init/init.c -o obj/kern/init/init.o。生成其它.o文件的过程类似，不再赘述
	|   |   |   | gcc命令的参数：
	|   |   |   |   | -ggdb                生成可供gdb使用的调试信息。
	|   |   |   |   | -m32                 生成适用于32位环境的代码。
	|   |   |   |   | -gstabs              生成stabs格式的调试信息。
	|   |   |   |   | -nostdinc            不使用标准库。
	|   |   |   |   | -fno-stack-protector 不生成用于检测缓冲区溢出的代码。
	|   |   |   |   | -Os                  为减小代码大小而进行优化。
	|   |   |   |   | -I<dir>              添加搜索头文件的路径
	|   | 
	|   | @echo + ld $@
	|   | $(V)$(LD) $(LDFLAGS) -T tools/kernel.ld -o $@ $(KOBJS)
	|   |   | 实际执行的命令是ld -m    elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel  obj/kern/init/init.o obj/kern/libs/readline.o obj/kern/libs/stdio.o obj/kern/debug/kdebug.o obj/kern/debug/kmonitor.o obj/kern/debug/panic.o obj/kern/driver/clock.o obj/kern/driver/console.o obj/kern/driver/intr.o obj/kern/driver/picirq.o obj/kern/trap/trap.o obj/kern/trap/trapentry.o obj/kern/trap/vectors.o obj/kern/mm/pmm.o  obj/libs/printfmt.o obj/libs/string.o
	|   |   | ld 的参数
	|   |   |   | -m <emulation>  模拟为i386上的连接器
	|   |   |   | -nostdlib       不使用标准库
	|   |   |   | -N              设置代码段和数据段均可读写
	|   |   |   | -e <entry>      指定入口
	|   |   |   | -Ttext          指定代码段开始位置
	|   |   |   | -T <scriptfile> 让连接器使用指定的脚本
	|   | @$(OBJDUMP) -S $@ > $(call asmfile,kernel)
	|   | | makefile的几条指令中有@前缀的都不必需
	|   | @$(OBJDUMP) -t $@ | $(SED) '1,/SYMBOL TABLE/d; s/ .* / /; /^$$/d' > $(call symfile,kernel)
	| 
	| > $(bootblock)
	|   | (bootblock): $(call toobj,$(bootfiles)) | $(call totarget,sign)
	|   | > bootfiles
	|   |   | bootfiles = $(call listf_cc,boot)
	|   |   | $(foreach f,$(bootfiles),$(call cc_compile,$(f),$(CC),$(CFLAGS) -Os -nostdinc))
	|   |   |   | 以生成bootasm.o为例，执行的命令是gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootasm.S -o obj/boot/bootasm.o。生成其它.o文件也类似，不再赘述
	|   | 
	|   | > sign
	|   |   | $(call add_files_host,tools/sign.c,sign,sign)
	|   |   |   | gcc -Itools/ -g -Wall -O2 -c tools/sign.c -o obj/sign/tools/sign.o
	|   |   | $(call create_target_host,sign,sign)
	|   |   |   | gcc -g -Wall -O2 obj/sign/tools/sign.o -o bin/sign
	|   | 
	|   | @echo + ld $@
	|   | $(V)$(LD) $(LDFLAGS) -N -e start -Ttext 0x7C00 $^ -o $(call toobj,bootblock)
	|   |   | ld -m    elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o
	|   | @$(OBJDUMP) -S $(call objfile,bootblock) > $(call asmfile,bootblock)
	|   | @$(OBJDUMP) -t $(call objfile,bootblock) | $(SED) '1,/SYMBOL TABLE/d; s/ .* / /; /^$$/d' > $(call symfile,bootblock)
	|   | @$(OBJCOPY) -S -O binary $(call objfile,bootblock) $(call outfile,bootblock)
	|   | @$(call totarget,sign) $(call outfile,bootblock) $(bootblock)
	| 
	| $(V)dd if=/dev/zero of=$@ count=10000
	|   | dd if=/dev/zero of=bin/ucore.img count=10000
	| $(V)dd if=$(bootblock) of=$@ conv=notrunc
	|   | dd if=bin/bootblock of=bin/ucore.img conv=notrunc
	| $(V)dd if=$(kernel) of=$@ seek=1 conv=notrunc
	|   | dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc

###1.2一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？
‘扇区大小512个字节，其中结尾两个字节分别为0x55和0xAA’
