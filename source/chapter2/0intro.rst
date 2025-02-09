引言
================================

本章导读
---------------------------------


**批处理系统** (Batch System) 出现于计算资源匮乏的年代，其核心思想是：
将多个程序打包到一起输入计算机；当一个程序运行结束后，计算机会 *自动* 执行下一个程序。

应用程序难免会出错，如果一个程序的错误导致整个操作系统都无法运行，那就太糟糕了。
*保护* 操作系统不受出错程序破坏的机制被称为 **特权级** (Privilege) 机制，
它实现了用户态和内核态的隔离。

本章在上一章的基础上，让我们的 OS 内核能以批处理的形式一次运行多个应用程序，同时利用特权级机制，
令 OS 不因出错的用户态程序而崩溃。

本章首先为批处理操作系统设计用户程序，再阐述如何将这些应用程序链接到内核中，最后介绍如何利用特权级机制处理 Trap.

实践体验
---------------------------

本章我们引入了用户程序。为了将内核与应用解耦，我们将二者分成了两个仓库，分别是存放内核程序的 ``rCore-Tutorial-Code-20xxx`` （下称代码仓库，最后几位 x 表示学期）与存放用户程序的 ``rCore-Tutorial-Test-20xxx`` （下称测例仓库）。 你首先需要进入代码仓库文件夹并 clone 用户程序仓库（如果已经执行过该步骤则不需要再重复执行）：

.. code-block:: console

   $ git clone https://github.com/LearningOS/rCore-Tutorial-Code-2025S.git
   $ cd rCore-Tutorial-Code-2025S
   $ git checkout ch2
   $ git clone https://github.com/LearningOS/rCore-Tutorial-Test-2025S.git user

上面的指令会将测例仓库克隆到代码仓库下并命名为 ``user`` ，注意 ``/user`` 在代码仓库的 ``.gitignore`` 文件中，因此不会出现 ``.git`` 文件夹嵌套的问题，并且你在代码仓库进行 checkout 操作时也不会影响测例仓库的内容。

在 qemu 模拟器上运行本章代码：

.. code-block:: console

   $ cd os
   $ make run LOG=INFO

批处理系统自动加载并运行了所有的用户程序，尽管某些程序出错了：

.. code-block::

      [rustsbi] RustSBI version 0.3.1, adapting to RISC-V SBI v1.0.0
   .______       __    __      _______.___________.  _______..______   __
   |   _  \     |  |  |  |    /       |           | /       ||   _  \ |  |
   |  |_)  |    |  |  |  |   |   (----`---|  |----`|   (----`|  |_)  ||  |
   |      /     |  |  |  |    \   \       |  |      \   \    |   _  < |  |
   |  |\  \----.|  `--'  |.----)   |      |  |  .----)   |   |  |_)  ||  |
   | _| `._____| \______/ |_______/       |__|  |_______/    |______/ |__|
   [rustsbi] Implementation     : RustSBI-QEMU Version 0.2.0-alpha.2
   [rustsbi] Platform Name      : riscv-virtio,qemu
   [rustsbi] Platform SMP       : 1
   [rustsbi] Platform Memory    : 0x80000000..0x88000000
   [rustsbi] Boot HART          : 0
   [rustsbi] Device Tree Region : 0x87e00000..0x87e010de
   [rustsbi] Firmware Address   : 0x80000000
   [rustsbi] Supervisor Address : 0x80200000
   [rustsbi] pmp01: 0x00000000..0x80000000 (-wr)
   [rustsbi] pmp02: 0x80000000..0x80200000 (---)
   [rustsbi] pmp03: 0x80200000..0x88000000 (xwr)
   [rustsbi] pmp04: 0x88000000..0x00000000 (-wr)
   [kernel] Hello, world!
   [ INFO] [kernel] .data [0x8020b000, 0x80228000)
   [ WARN] [kernel] boot_stack top=bottom=0x80238000, lower_bound=0x80228000
   [ERROR] [kernel] .bss [0x80238000, 0x80239000)
   [kernel] num_app = 7
   [kernel] app_0 [0x8020b048, 0x8020f0f0)
   [kernel] app_1 [0x8020f0f0, 0x80213198)
   [kernel] app_2 [0x80213198, 0x80217240)
   [kernel] app_3 [0x80217240, 0x8021b2e8)
   [kernel] app_4 [0x8021b2e8, 0x8021f390)
   [kernel] app_5 [0x8021f390, 0x80223438)
   [kernel] app_6 [0x80223438, 0x802274e0)
   [kernel] Loading app_0
   [kernel] PageFault in application, kernel killed it.
   [kernel] Loading app_1
   [kernel] IllegalInstruction in application, kernel killed it.
   [kernel] Loading app_2
   [kernel] IllegalInstruction in application, kernel killed it.
   [kernel] Loading app_3
   Hello, world from user mode program!
   [kernel] Loading app_4
   power_3 [10000/200000]
   power_3 [20000/200000]
   power_3 [30000/200000]
   power_3 [40000/200000]
   power_3 [50000/200000]
   power_3 [60000/200000]
   power_3 [70000/200000]
   power_3 [80000/200000]
   power_3 [90000/200000]
   power_3 [100000/200000]
   power_3 [110000/200000]
   power_3 [120000/200000]
   power_3 [130000/200000]
   power_3 [140000/200000]
   power_3 [150000/200000]
   power_3 [160000/200000]
   power_3 [170000/200000]
   power_3 [180000/200000]
   power_3 [190000/200000]
   power_3 [200000/200000]
   3^200000 = 871008973(MOD 998244353)
   Test power_3 OK!
   [kernel] Loading app_5
   power_5 [10000/140000]
   power_5 [20000/140000]
   power_5 [30000/140000]
   power_5 [40000/140000]
   power_5 [50000/140000]
   power_5 [60000/140000]
   power_5 [70000/140000]
   power_5 [80000/140000]
   power_5 [90000/140000]
   power_5 [100000/140000]
   power_5 [110000/140000]
   power_5 [120000/140000]
   power_5 [130000/140000]
   power_5 [140000/140000]
   5^140000 = 386471875(MOD 998244353)
   Test power_5 OK!
   [kernel] Loading app_6
   power_7 [10000/160000]
   power_7 [20000/160000]
   power_7 [30000/160000]
   power_7 [40000/160000]
   power_7 [50000/160000]
   power_7 [60000/160000]
   power_7 [70000/160000]
   power_7 [80000/160000]
   power_7 [90000/160000]
   power_7 [100000/160000]
   power_7 [110000/160000]
   power_7 [120000/160000]
   power_7 [130000/160000]
   power_7 [140000/160000]
   power_7 [150000/160000]
   power_7 [160000/160000]
   7^160000 = 667897727(MOD 998244353)
   Test power_7 OK!
   All applications completed!

本章代码树
-------------------------------------------------

.. code-block::

   ── os
   │   ├── Cargo.toml
   │   ├── Makefile (修改：构建内核之前先构建应用)
   │   ├── build.rs (新增：生成 link_app.S 将应用作为一个数据段链接到内核)
   │   └── src
   │       ├── batch.rs(新增：实现了一个简单的批处理系统)
   │       ├── console.rs
   │       ├── entry.asm
   │       ├── lang_items.rs
   │       ├── link_app.S(构建产物，由 os/build.rs 输出)
   │       ├── linker.ld
   │       ├── logging.rs
   │       ├── main.rs(修改：主函数中需要初始化 Trap 处理并加载和执行应用)
   │       ├── sbi.rs
   │       ├── sync(新增：包装了RefCell，暂时不用关心)
   │       │   ├── mod.rs
   │       │   └── up.rs
   │       ├── syscall(新增：系统调用子模块 syscall)
   │       │   ├── fs.rs(包含文件 I/O 相关的 syscall)
   │       │   ├── mod.rs(提供 syscall 方法根据 syscall ID 进行分发处理)
   │       │   └── process.rs(包含任务处理相关的 syscall)
   │       └── trap(新增：Trap 相关子模块 trap)
   │           ├── context.rs(包含 Trap 上下文 TrapContext)
   │           ├── mod.rs(包含 Trap 处理入口 trap_handler)
   │           └── trap.S(包含 Trap 上下文保存与恢复的汇编代码)
   └── user(新增：应用测例保存在 user 目录下)
      ├── Cargo.toml
      ├── Makefile
      └── src
         ├── bin(基于用户库 user_lib 开发的应用，每个应用放在一个源文件中)
         │   ├── ...
         ├── console.rs
         ├── lang_items.rs
         ├── lib.rs(用户库 user_lib)
         ├── linker.ld(应用的链接脚本)
         └── syscall.rs(包含 syscall 方法生成实际用于系统调用的汇编指令，
                        各个具体的 syscall 都是通过 syscall 来实现的)

   cloc os
   -------------------------------------------------------------------------------
   Language                     files          blank        comment           code
   -------------------------------------------------------------------------------
   Rust                            14             62             21            435
   Assembly                         3              9             16            106
   make                             1             12              4             36
   TOML                             1              2              1              9
   -------------------------------------------------------------------------------
   SUM:                            19             85             42            586
   -------------------------------------------------------------------------------
