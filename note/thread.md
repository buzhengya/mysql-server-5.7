- [[#线程分类|线程分类]]
- [[#master thread|master thread]]
- [[#srv_sys_t|srv_sys_t]]
- [[#srv_slot_t|srv_slot_t]]


# 临时信息
### 线程分类
``` c++
enum srv_thread_type {
	SRV_NONE,			/*!< None */
	SRV_WORKER,			/*!< threads serving parallelized
					queries and queries released from
					lock wait */
	SRV_PURGE,			/*!< Purge coordinator thread */
	SRV_MASTER			/*!< the master thread, (whose type
					number must be biggest) */
};
```

### master thread
declare
```c++
os_thread_ret_t
DECLARE_THREAD(srv_master_thread)(
/*==============================*/
	void*	arg);	/*!< in: a dummy parameter required by
			os_thread_create */
```
### srv_sys_t
```c++
/** The server system struct */
struct srv_sys_t{
	ib_mutex_t	tasks_mutex;		/*!< variable protecting the task 变量的锁
						tasks queue */
	UT_LIST_BASE_NODE_T(que_thr_t)
			tasks;			/*!< task queue 查询task队列 */

	ib_mutex_t	mutex;			/*!< variable protecting the 下面其它变量的锁
						fields below. */
	ulint		n_sys_threads;		/*!< size of the sys_threads
						array */

	srv_slot_t*	sys_threads;		/*!< server thread table 每个thread对应一个srv_slot_t, master thread, purge thread 固定1个，因此固定在1、2位置 */

	ulint		n_threads_active[SRV_MASTER + 1]; // 4种类型的线程的active状态的数量 
						/*!< number of threads active
						in a thread class */

	srv_stats_t::ulint_ctr_1_t
			activity_count;		/*!< For tracking server
						activity  服务的各thread调度运行时+1 */
};

static srv_sys_t*	srv_sys	= NULL; // 全局唯一的数据结构
```
### srv_slot_t 
`srv_reserve_slot`：线程创建时，创建其对应的`srv_slot_t`
`srv_suspend_thread`：挂起thread
`srv_release_threads`：将thread从suspend release。继续运行
`srv_free_slot`：释放thread及`srv_free_slot`

### 主要线程的描述
`master_threads`：只有一个thread，将缓冲区的数据异步的刷入到磁盘中。
`io_thread`：异步处理IO请求，`show variables like '%innodb%io_thread%'`
`purge_thread`：回收并使用undo页。`show variables like '%innodb%purge_thread%'`
Q：处理请求的thread在哪里？innodb提供请求处理API，请求的处理线程应该是由mysql创建和调度。
`page_cleaner_thread`：
### 主要内存缓冲区描述
`show engine innodb status` ：显示db的thread、buffer_pool等状态信息。
数据页：
索引页：
插入缓冲区：
锁信息：
重做日志：
`LRU List`：
`Flush List`：
`Free List`：