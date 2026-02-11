## Write 0xCAFEBABE to the eax register

### Loader

Why assembly code => C requires a stack.

The file that we will work with is called : `loader.s`

The file loader.s can be compiled into 32-bit ELF object file with the following command:

```bash
nasm -f elf32 loader.s
```

### Linking the kernel

The Code must now be linked to produce an executable file.

=> `GRUB` => Load the kernel at memory address >= 0x00100000 (1 MB)

+ because addresses <= 1 MB are userd by GRUB itself, BIOS and memory-mapped I/O. (linker script needed is : `GNU LD`)

The linker script is a file called `link.ld`

```bash
ld -T link.ld -melf_i386 loader.o -o kernel.elf
```

### Building an ISO Image

We will create the kernel ISO image with program `genisoimage` => Folder and correct places.

```bash
mkdir -p iso/boot/grub                        # the folder structure
cp stage2_eltorito iso/boot/grub              # copy the bootloader
cp kernel.elf iso/boot/                       # copy the kernel
```

A Configuration file `menu.lst` for GRUB. (Tells GRUB where the kernel is located and configures some options)

+ the content of the iso folder

```
iso
└── boot
    ├── grub
    │   ├── menu.lst
    │   └── stage2_eltorito
    └── kernel.elf
```

+ ISO image generation command :

```bash
genisoimage -R                              \
            -b boot/grub/stage2_eltorito    \
            -no-emul-boot                   \
            -boot-load-size 4               \
            -A os                           \
            -input-charset utf8             \
            -quiet                          \
            -boot-info-table                \
            -o os.iso                       \
            iso
```

+ Running Bochs (configuration and emulation)

```text
megs:            32
display_library: sdl
romimage:        file=/usr/share/bochs/BIOS-bochs-latest
vgaromimage:     file=/usr/share/bochs/VGABIOS-lgpl-latest
ata0-master:     type=cdrom, path=os.iso, status=inserted
boot:            cdrom
log:             bochslog.txt
clock:           sync=realtime, time0=local
cpu:             count=1, ips=1000000
```

```bash
bochs -f bochsrc.txt -q
```

```bash
qemu-system-i386 -cdrom os.iso -m 32M -boot d -d cpu,int,exec -D qemu-log.txt
```
