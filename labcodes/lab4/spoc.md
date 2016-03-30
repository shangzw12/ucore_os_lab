#进程控制块的数据结构
	struct proc_struct {
	    enum proc_state state;                      // Process state
	    int pid;                                    // Process ID
	    int runs;                                   // the running times of Proces
	    uintptr_t kstack;                           // Process kernel stack
	    volatile bool need_resched;                 // bool value: need to be rescheduled to release CPU?
	    struct proc_struct *parent;                 // the parent process
	    struct mm_struct *mm;                       // Process's memory management field
	    struct context context;                     // Switch here to run process
	    struct trapframe *tf;                       // Trap frame for current interrupt
	    uintptr_t cr3;                              // CR3 register: the base addr of Page Directroy Table(PDT)
	    uint32_t flags;                             // Process flag
	    char name[PROC_NAME_LEN + 1];               // Process name
	    list_entry_t list_link;                     // Process link list 
	    list_entry_t hash_link;                     // Process hash list
	};
	主要字段如上面代码显示
	包括进程的状态/进程ID/进程运行时间等

####有下面函数对其进行了修改

	alloc_proc(void)//用于进程控制块的初始化，用于唯一标示进程
	char *	set_proc_name(struct proc_struct *proc, const char *name)  //设置进程名
	get_pid()// 给进程设置唯一的pid
	proc_run() //用于执行结构体中对应的进程
	// setup_kstack - alloc pages with size KSTACKPAGE as process kernel stack
	static int setup_kstack(struct proc_struct *proc) //为进程设置堆栈
	// put_kstack - free the memory space of process kernel stack
	static void put_kstack(struct proc_struct *proc) //释放进程的堆栈
	proc_init()//对idle_proc进行初始化
	
####有下面函数用到了他
	上面所有的要修改他的函数
	get_proc_name()//获取进程的name
	proc_run() //执行结构体对应的进程
	hash_proc()//为了search 进程的高效，先对进程的pid进行hash
	find_proc()//获取制定pid的进程
	copy_mm() //拷贝和当前进程内存几乎一样的内存
	copy_thread()//同上
	do_fork()//fork一个新的进程