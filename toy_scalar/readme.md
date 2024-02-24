


## add toy_scalar machine support

在这个repo上，这个步骤里说明的修改都已经完成。

### 1. hw/riscv/Kconfig

在这个文件尾部加上这段代码：

    config SCALAR_TOY
        bool
        select RISCV_NUMA
        select HTIF
        select RISCV_ACLINT
        select SIFIVE_PLIC
        select SERIAL

### 2. hw/riscv/meson.build

在:
    
    riscv_ss.add(when: 'CONFIG_SPIKE', if_true: files('spike.c'))

之后，加上:
        
    riscv_ss.add(when: 'CONFIG_SCALAR_TOY', if_true: files('scalar_toy.c'))

这里注册了socalar_toy.c这个配置文件

### 3. hw/riscv/scalar_toy.c

这个文件是完全新增的，由hw/riscv/spike.c修改而来，描述了scalar_toy这个machine对应的地址映射和挂载的外设。具体代码参见本repo对应位置。

### 4. configs/devices/riscv32-softmmu

在这个文件尾部添加：
    
    CONFIG_SCALAR_TOY=y

### 5. configs/devices/riscv64-softmmu

在这个文件尾部添加：

    CONFIG_SCALAR_TOY=y




## configure & compile

这一步是pull这个repo以后需要执行的流程，完成qemu的构建。


创建并进入build文件夹:

    mkdir build
    cd build

使用

    ../configure --help

命令可以查看支持的配置选项。对于riscv我们使用如下配置:

    ../configure --target-list=riscv64-softmmu,riscv32-softmmu --enable-trace-backends=simple

这个配置制定了QEMU模拟的目标硬件为rv64和rv32，会生成两个qemu程序，分别对应这两个硬件。

随后使用make进行编译，可以指定多核加速:

    make -j64

编译完成后，build目录下的:

    qemu-system-riscv32
    qemu-system-riscv64

就是QEMU的可执行程序，可以尝试执行来查看支持的machine中是否有scalar_toy。

    ./qemu-system-riscv32 -machine help

## run

这是一个运行QEMU的例子:

    ./qemu-system-riscv32 \
        -nographic -m 2048 -smp 1 \
        -M scalar_toy \
        -singlestep -d tid,cpu,exec -D log%d \
        -device loader,file=/tools/zdchen/riscv-proj/hello_world/build/dhrystone.elf,cpu-num=0
