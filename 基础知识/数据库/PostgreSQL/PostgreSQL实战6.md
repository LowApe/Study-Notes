# 并行查询



# 1 并行查询相关配置参数

| 参数                                     | 介绍                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| max_worker_processes(integer)            | **设置系统支持的最大后台进程数**，默认 8 ，如果有备库，备库上此参数必须大于或等于主库上的参数配置值，此参数调整后需重启数据库生效。 |
| max_parallel_workers(integer)            | **设置系统支持的并行查询进程数**                             |
| max_parallel_workers_per_gather(integer) | 设置允许启动的并行进程的进程数，默认值为2，设置 0 禁用并行查询 |
|                                          |                                                              |
|                                          |                                                              |
|                                          |                                                              |
|                                          |                                                              |
|                                          |                                                              |
|                                          |                                                              |
|                                          |                                                              |
|                                          |                                                              |



# 2 并行扫描

- 并行顺序扫描
- 并行索引扫描
- 并行 index-only 扫描
- 并行 bitmap heap 扫描场景



# 相关链接