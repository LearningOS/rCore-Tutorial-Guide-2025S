chapter3练习
=======================================

编程作业
--------------------------------------

获取任务信息
++++++++++++++++++++++++++

在 ch3 中，我们的系统已经能够支持多个任务分时轮流运行，我们希望引入一个新的系统调用 ``sys_trace``（ID 为 410）用来追踪当前任务系统调用的历史信息，并做对应的修改。定义如下。

.. code-block:: rust

    fn sys_trace(_trace_request: usize, _id: usize, _data: usize) -> isize


- 调用规范：
   - 这个系统调用有三种功能，根据 ``trace_request`` 的值不同，执行不同的操作：
   - 如果 ``trace_request`` 为 0，则 ``id`` 应被视作 ``*const u8`` ，表示读取当前任务 ``id`` 地址处一个字节的无符号整数值。此时应忽略 ``data`` 参数。返回值为 ``id`` 地址处的值。
   - 如果 ``trace_request`` 为 1，则 ``id`` 应被视作 ``*const u8`` ，表示写入 ``data`` （作为 ``u8``，即只考虑最低位的一个字节）到该用户程序 ``id`` 地址处。返回值应为0。
   - 如果 ``trace_request`` 为 2，表示查询当前任务调用编号为 ``id`` 的系统调用的次数，返回值为这个调用次数。**本次调用也计入统计** 。
   - 否则，忽略其他参数，返回值为 -1。

- 说明：
   - 你可能会注意到，这个调用的读写并不安全，使用不当可能导致崩溃。这是因为在下一章节实现地址空间之前，系统中缺乏隔离机制。所以我们 **不要求你实现安全检查机制，只需通过测试用例即可** 。
   - 你还可能注意到，这个系统调用读写本任务内存的功能并不是很有用。这是因为作业的灵感来源 syscall 主要依靠 trace 功能追踪其他任务的信息，但在本章节我们还没有进程、线程等概念，所以简化了操作，只要求追踪自身的信息。
- 提示：
    - 大胆修改已有框架！除了配置文件，你几乎可以随意修改已有框架的内容。
    - 系统调用次数可以考虑在内核态的 ``syscall`` 函数中统计。
    - 可以扩展 ``TaskManagerInner`` 中的结构来维护新的信息。
    - 不要害怕使用 ``unsafe`` 做类型转换，这在内核处理用户调用时是不可避免的。
    - 在实现时，可以把系统调用参数中前缀的下划线去掉，这样更清晰。实验框架之所以这么写，是因为在没有使用对应参数的情况下，Rust 推荐使用下划线前缀以避免警告。

实验要求
+++++++++++++++++++++++++++++++++++++++++

- 完成分支: ch3。

- 实验目录要求

.. code-block::

   ├── os(内核实现)
   │   ├── Cargo.toml(配置文件)
   │   └── src(所有内核的源代码放在 os/src 目录下)
   │       ├── main.rs(内核主函数)
   │       └── ...
   ├── reports (不是 report)
   │   ├── lab1.md/pdf
   │   └── ...
   ├── ...


- 通过所有测例：

   CI 使用的测例与本地相同，测试中，user 文件夹及其它与构建相关的文件将被替换，请不要试图依靠硬编码通过测试。

   默认情况下，makefile 仅编译基础测例 (``BASE=1``)，即无需修改框架即可正常运行的测例。
   你需要在编译时指定 ``BASE=0`` 控制框架仅编译实验测例（在 os 目录执行 ``make run BASE=0``），
   或指定 ``BASE=2`` 控制框架同时编译基础测例和实验测例。

- 如果本地在线编译访问 github 遇到问题，可以对 os/Cargo.toml 中的依赖进行如下替换：

   .. code-block:: shell
      
      riscv = { git = "https://gitee.com/rcore-os/riscv", features = ["inline-asm"] }
      virtio-drivers = { git = "https://gitee.com/rcore-os/virtio-drivers", rev = "4ee80e5" }


- 本地离线运行 CI 脚本的方法（注意 CI 脚本会修改代码仓库的文件，请在运行前暂存改动）：

   .. code-block:: shell

      git clone git@git.tsinghua.edu.cn:os-lab/2024A/public/rcore-tutorial-checker-2024A.git ci-user
      git clone git@git.tsinghua.edu.cn:os-lab/2024A/public/rcore-tutorial-test-2024A.git ci-user/user
      cd ci-user && make test CHAPTER=$ID OFFLINE=1


.. note::

    你的实现只需且必须通过测例，建议读者感到困惑时先检查测例。


简答作业
--------------------------------------------

1. 正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。
   请同学们可以自行测试这些内容（运行 `三个 bad 测例 (ch2b_bad_*.rs) <https://github.com/LearningOS/rCore-Tutorial-Test-2024A/tree/master/src/bin>`_ ），
   描述程序出错行为，同时注意注明你使用的 sbi 及其版本。

2. 深入理解 `trap.S <https://github.com/LearningOS/rCore-Tutorial-Code-2024A/blob/ch3/os/src/trap/trap.S>`_
   中两个函数 ``__alltraps`` 和 ``__restore`` 的作用，并回答如下问题:

   1. L40：刚进入 ``__restore`` 时，``a0`` 代表了什么值。请指出 ``__restore`` 的两种使用情景。

   2. L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。

      .. code-block:: riscv

         ld t0, 32*8(sp)
         ld t1, 33*8(sp)
         ld t2, 2*8(sp)
         csrw sstatus, t0
         csrw sepc, t1
         csrw sscratch, t2

   3. L50-L56：为何跳过了 ``x2`` 和 ``x4``？

      .. code-block:: riscv

         ld x1, 1*8(sp)
         ld x3, 3*8(sp)
         .set n, 5
         .rept 27
            LOAD_GP %n
            .set n, n+1
         .endr

   4. L60：该指令之后，``sp`` 和 ``sscratch`` 中的值分别有什么意义？

      .. code-block:: riscv

         csrrw sp, sscratch, sp

   5. ``__restore``：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？

   6. L13：该指令之后，``sp`` 和 ``sscratch`` 中的值分别有什么意义？

      .. code-block:: riscv

         csrrw sp, sscratch, sp

   7. 从 U 态进入 S 态是哪一条指令发生的？

报告要求
-------------------------------

- 简单总结你实现的功能（200字以内，不要贴代码）。
- 完成问答题。
- 加入 :doc:`/honorcode` 的内容。否则，你的提交将视作无效，本次实验的成绩将按“0”分计。
- 推荐markdown文档格式。
- (optional) 你对本次实验设计及难度/工作量的看法，以及有哪些需要改进的地方，欢迎畅所欲言。
