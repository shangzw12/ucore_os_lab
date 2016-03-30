#���̿��ƿ�����ݽṹ
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
	��Ҫ�ֶ������������ʾ
	�������̵�״̬/����ID/��������ʱ���

####�����溯������������޸�

	alloc_proc(void)//���ڽ��̿��ƿ�ĳ�ʼ��������Ψһ��ʾ����
	char *	set_proc_name(struct proc_struct *proc, const char *name)  //���ý�����
	get_pid()// ����������Ψһ��pid
	proc_run() //����ִ�нṹ���ж�Ӧ�Ľ���
	// setup_kstack - alloc pages with size KSTACKPAGE as process kernel stack
	static int setup_kstack(struct proc_struct *proc) //Ϊ�������ö�ջ
	// put_kstack - free the memory space of process kernel stack
	static void put_kstack(struct proc_struct *proc) //�ͷŽ��̵Ķ�ջ
	proc_init()//��idle_proc���г�ʼ��
	
####�����溯���õ�����
	�������е�Ҫ�޸����ĺ���
	get_proc_name()//��ȡ���̵�name
	proc_run() //ִ�нṹ���Ӧ�Ľ���
	hash_proc()//Ϊ��search ���̵ĸ�Ч���ȶԽ��̵�pid����hash
	find_proc()//��ȡ�ƶ�pid�Ľ���
	copy_mm() //�����͵�ǰ�����ڴ漸��һ�����ڴ�
	copy_thread()//ͬ��
	do_fork()//forkһ���µĽ���