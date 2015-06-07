## Revision: 8e099d1e8be3f598dcefd04d3cd5eb3673d4e098 ##

tsan-dev revision: 0db4196a68779d0f3c714bb092a09a93a52b8557

```
==================================================================
ThreadSanitizer: data-race in cache_reap

Read of size 4 by thread T298 (K295):
 [<ffffffff81222be1>] cache_reap+0x141/0x350 /mm/slab.c:4075
 [<ffffffff810aaf3e>] process_one_work+0x2ee/0x730 /kernel/workqueue.c:2027
 [<ffffffff810abe9b>] worker_thread+0xcb/0x740 /kernel/workqueue.c:2160
 [<ffffffff810b5a2b>] kthread+0x14b/0x170 /kernel/kthread.c:207
 [<ffffffff81dcb53c>] ret_from_fork+0x7c/0xb0 /arch/x86/kernel/entry_64.S:347
DBG: cpu = ffff88041fd95df0

Previous write of size 4 by thread T1 (K1):
 [<     inlined    >] kmem_cache_alloc+0x8c9/0x9b0 cache_alloc_refill /mm/slab.c:2944
 [<     inlined    >] kmem_cache_alloc+0x8c9/0x9b0 ____cache_alloc /mm/slab.c:3098
 [<     inlined    >] kmem_cache_alloc+0x8c9/0x9b0 __do_cache_alloc /mm/slab.c:3356
 [<     inlined    >] kmem_cache_alloc+0x8c9/0x9b0 slab_alloc /mm/slab.c:3395
 [<ffffffff81228209>] kmem_cache_alloc+0x8c9/0x9b0 /mm/slab.c:3557
 [<     inlined    >] __kernfs_new_node+0x75/0x150 kmem_cache_zalloc /include/linux/slab.h:629
 [<ffffffff812f1db5>] __kernfs_new_node+0x75/0x150 /fs/kernfs/dir.c:526
 [<ffffffff812f36f7>] kernfs_new_node+0x57/0x90 /fs/kernfs/dir.c:558
 [<ffffffff812f3ae1>] kernfs_create_dir_ns+0x41/0xc0 /fs/kernfs/dir.c:771
 [<ffffffff812f732b>] sysfs_create_dir_ns+0x6b/0xf0 /fs/sysfs/dir.c:55
 [<     inlined    >] kobject_add_internal+0x166/0x5b0 create_dir /lib/kobject.c:71
 [<ffffffff814f20f6>] kobject_add_internal+0x166/0x5b0 /lib/kobject.c:229
 [<     inlined    >] kobject_add+0x86/0xe0 kobject_add_varg /lib/kobject.c:354
 [<ffffffff814f27d6>] kobject_add+0x86/0xe0 /lib/kobject.c:399
 [<ffffffff8176ef6c>] device_add+0x3dc/0x9b0 /drivers/base/core.c:1010
 [<ffffffff8176f561>] device_register+0x21/0x30 /drivers/base/core.c:1128
 [<ffffffff815e3c43>] tty_register_device_attr+0x193/0x3d0 /drivers/tty/tty_io.c:3173
 [<     inlined    >] tty_register_driver+0x240/0x3c0 tty_register_device /drivers/tty/tty_io.c:3100
 [<ffffffff815e4100>] tty_register_driver+0x240/0x3c0 /drivers/tty/tty_io.c:3360
 [<ffffffff81db69fe>] vty_init+0x212/0x23c /drivers/tty/vt/vt.c:3064
 [<ffffffff82336e88>] tty_init+0x174/0x17e /drivers/tty/tty_io.c:3589
 [<ffffffff82338509>] chr_dev_init+0xf2/0x102 /drivers/char/mem.c:904
 [<ffffffff8100032d>] do_one_initcall+0xbd/0x220 /init/main.c:794
 [<     inlined    >] kernel_init_freeable+0x2bb/0x382 do_initcall_level /init/main.c:860
 [<     inlined    >] kernel_init_freeable+0x2bb/0x382 do_initcalls /init/main.c:868
 [<     inlined    >] kernel_init_freeable+0x2bb/0x382 do_basic_setup /init/main.c:887
 [<ffffffff822fd81a>] kernel_init_freeable+0x2bb/0x382 /init/main.c:1008
 [<ffffffff810004a6>] kernel_init+0x16/0x150 /init/main.c:938
 [<ffffffff81dcb53c>] ret_from_fork+0x7c/0xb0 /arch/x86/kernel/entry_64.S:347
DBG: cpu = ffff88041fc95df0

DBG: addr: ffff8801e5988720
DBG: first offset: 0, second offset: 0
DBG: T298 clock: {T298: 2620, T1: 28259295}
DBG: T1 clock: {T1: 28273825}
==================================================================
```

```
==================================================================
ThreadSanitizer: data-race in drain_array

Read of size 4 by thread T298 (K295):
 [<ffffffff81221f56>] drain_array+0x36/0x160 /mm/slab.c:4008 (discriminator 1)
 [<ffffffff81222bd9>] cache_reap+0x139/0x350 /mm/slab.c:4073
 [<ffffffff810aaf3e>] process_one_work+0x2ee/0x730 /kernel/workqueue.c:2027
 [<ffffffff810abe9b>] worker_thread+0xcb/0x740 /kernel/workqueue.c:2160
 [<ffffffff810b5a2b>] kthread+0x14b/0x170 /kernel/kthread.c:207
 [<ffffffff81dcb53c>] ret_from_fork+0x7c/0xb0 /arch/x86/kernel/entry_64.S:347
DBG: cpu = ffff88041fd95df0

Previous write of size 4 by thread T6 (K6):
 [<ffffffff812209d5>] transfer_objects+0x85/0xd0 /mm/slab.c:956
 [<     inlined    >] kmem_cache_alloc_node+0x43a/0x9f0 cache_alloc_refill /mm/slab.c:2933
 [<     inlined    >] kmem_cache_alloc_node+0x43a/0x9f0 ____cache_alloc /mm/slab.c:3098
 [<     inlined    >] kmem_cache_alloc_node+0x43a/0x9f0 slab_alloc_node /mm/slab.c:3324
 [<ffffffff8122390a>] kmem_cache_alloc_node+0x43a/0x9f0 /mm/slab.c:3595
 [<     inlined    >] copy_process.part.43+0x14c/0x2d70 alloc_task_struct_node /kernel/fork.c:130
 [<     inlined    >] copy_process.part.43+0x14c/0x2d70 dup_task_struct /kernel/fork.c:305
 [<ffffffff8107aedc>] copy_process.part.43+0x14c/0x2d70 /kernel/fork.c:1193
 [<     inlined    >] do_fork+0x100/0x4d0 copy_process /kernel/fork.c:1163
 [<ffffffff8107dd20>] do_fork+0x100/0x4d0 /kernel/fork.c:1607
 [<ffffffff8107e125>] kernel_thread+0x35/0x50 /kernel/fork.c:1656
 [<ffffffff810a4a00>] __call_usermodehelper+0x40/0xe0 /kernel/kmod.c:333
 [<ffffffff810aaf3e>] process_one_work+0x2ee/0x730 /kernel/workqueue.c:2027
 [<ffffffff810abe9b>] worker_thread+0xcb/0x740 /kernel/workqueue.c:2160
 [<ffffffff810b5a2b>] kthread+0x14b/0x170 /kernel/kthread.c:207
 [<ffffffff81dcb53c>] ret_from_fork+0x7c/0xb0 /arch/x86/kernel/entry_64.S:347
DBG: cpu = ffff88041fc15df0

DBG: addr: ffff8801e5907400
DBG: first offset: 0, second offset: 0
DBG: T298 clock: {T298: 2999, T6: 3760856}
DBG: T6 clock: {T6: 3821168}
==================================================================
```

```
==================================================================
ThreadSanitizer: data-race in rb_insert_color

Read of size 8 by thread T6 (K6):
 [<     inlined    >] rb_insert_color+0x3d/0x3a0 __rb_insert /lib/rbtree.c:89
 [<ffffffff814f6f5d>] rb_insert_color+0x3d/0x3a0 /lib/rbtree.c:388
 [<ffffffff810cd800>] __enqueue_entity+0x70/0x80 /kernel/sched/fair.c:523
 [<     inlined    >] enqueue_task_fair+0x48c/0x1250 enqueue_entity /kernel/sched/fair.c:2804
 [<ffffffff810d060c>] enqueue_task_fair+0x48c/0x1250 /kernel/sched/fair.c:3948
 [<ffffffff810c32d5>] enqueue_task+0x35/0x60 /kernel/sched/core.c:845
 [<ffffffff810c67be>] activate_task+0x1e/0x20 /kernel/sched/core.c:860
 [<ffffffff810c9be1>] wake_up_new_task+0xd1/0x1b0 /kernel/sched/core.c:2098
 [<ffffffff8107dea6>] do_fork+0x286/0x4d0 /kernel/fork.c:1633
 [<ffffffff8107e125>] kernel_thread+0x35/0x50 /kernel/fork.c:1656
 [<ffffffff810a4a00>] __call_usermodehelper+0x40/0xe0 /kernel/kmod.c:333
 [<ffffffff810aaf3e>] process_one_work+0x2ee/0x730 /kernel/workqueue.c:2027
 [<ffffffff810abe9b>] worker_thread+0xcb/0x740 /kernel/workqueue.c:2160
 [<ffffffff810b5a2b>] kthread+0x14b/0x170 /kernel/kthread.c:207
 [<ffffffff81dcb53c>] ret_from_fork+0x7c/0xb0 /arch/x86/kernel/entry_64.S:347
DBG: cpu = ffff88041fd15df0

Previous write of size 8 by thread T1 (K1):
 [<     inlined    >] rb_insert_color+0x2fe/0x3a0 rb_set_parent_color /include/linux/rbtree_augmented.
h:107
 [<     inlined    >] rb_insert_color+0x2fe/0x3a0 __rb_insert /lib/rbtree.c:87
 [<ffffffff814f721e>] rb_insert_color+0x2fe/0x3a0 /lib/rbtree.c:388
 [<ffffffff810cd800>] __enqueue_entity+0x70/0x80 /kernel/sched/fair.c:523
 [<ffffffff810a77c1>] insert_work+0xd1/0x130 /kernel/workqueue.c:1263
 [<ffffffff810a7a4d>] __queue_work+0x22d/0x570 /kernel/workqueue.c:1378
 [<ffffffff810a8060>] queue_work_on+0x50/0x80 /kernel/workqueue.c:1403
 [<     inlined    >] call_usermodehelper_exec+0x1d4/0x1f0 queue_work /include/linux/workqueue.h:474
 [<ffffffff810a4d04>] call_usermodehelper_exec+0x1d4/0x1f0 /kernel/kmod.c:594
 [<ffffffff814f364e>] kobject_uevent_env+0x65e/0x6c0 /lib/kobject_uevent.c:351
 [<ffffffff814f36d3>] kobject_uevent+0x23/0x40 /lib/kobject_uevent.c:375
 [<     inlined    >] param_sysfs_init+0x1cc/0x1f5 kernel_add_sysfs_param /kernel/params.c:784
 [<     inlined    >] param_sysfs_init+0x1cc/0x1f5 param_sysfs_builtin /kernel/params.c:819
 [<ffffffff8231216a>] param_sysfs_init+0x1cc/0x1f5 /kernel/params.c:939
 [<ffffffff8100032d>] do_one_initcall+0xbd/0x220 /init/main.c:794
 [<     inlined    >] kernel_init_freeable+0x2bb/0x382 do_initcall_level /init/main.c:860
 [<     inlined    >] kernel_init_freeable+0x2bb/0x382 do_initcalls /init/main.c:868
 [<     inlined    >] kernel_init_freeable+0x2bb/0x382 do_basic_setup /init/main.c:887
 [<ffffffff822fd81a>] kernel_init_freeable+0x2bb/0x382 /init/main.c:1008
 [<ffffffff810004a6>] kernel_init+0x16/0x150 /init/main.c:938
 [<ffffffff81dcb53c>] ret_from_fork+0x7c/0xb0 /arch/x86/kernel/entry_64.S:347
DBG: cpu = 0

DBG: addr: ffff8801e34bf468
DBG: first offset: 0, second offset: 0
DBG: T6 clock: {T6: 1322733, T1: 9261340}
DBG: T1 clock: {T1: 9266129}
==================================================================
```

```
==================================================================
ThreadSanitizer: data-race in acpi_ns_attach_object

Write of size 1 by thread T1 (K1):
 [<ffffffff815b0bc3>] acpi_ns_attach_object+0x14b/0x18c /drivers/acpi/acpica/nsobject.c:184
 [<ffffffff815a0741>] acpi_install_notify_handler+0x13f/0x2ed /drivers/acpi/acpica/evxface.c:157
 [<     inlined    >] acpi_device_probe+0x117/0x181 acpi_device_install_notify_handler /drivers/acpi/s
can.c:946
 [<ffffffff81586931>] acpi_device_probe+0x117/0x181 /drivers/acpi/scan.c:991
 [<ffffffff817738a4>] really_probe+0xa4/0x350 /drivers/base/dd.c:302
 [<     inlined    >] __driver_attach+0xd9/0xe0 driver_probe_device /drivers/base/dd.c:399
 [<ffffffff81773cb9>] __driver_attach+0xd9/0xe0 /drivers/base/dd.c:477
 [<ffffffff817707fd>] bus_for_each_dev+0x9d/0xe0 /drivers/base/bus.c:311
 [<ffffffff8177319f>] driver_attach+0x2f/0x40 /drivers/base/dd.c:496
 [<ffffffff81772c50>] bus_add_driver+0x270/0x330 /drivers/base/bus.c:692
 [<ffffffff81774967>] driver_register+0xd7/0x1b0 /drivers/base/driver.c:167
 [<ffffffff81587585>] acpi_bus_register_driver+0x8b/0x9e /drivers/acpi/scan.c:1281
 [<ffffffff82335b7c>] acpi_ac_init+0x37/0x45 /drivers/acpi/ac.c:437
 [<ffffffff8100032d>] do_one_initcall+0xbd/0x220 /init/main.c:794
 [<     inlined    >] kernel_init_freeable+0x2bb/0x382 do_initcall_level /init/main.c:860
 [<     inlined    >] kernel_init_freeable+0x2bb/0x382 do_initcalls /init/main.c:868
 [<     inlined    >] kernel_init_freeable+0x2bb/0x382 do_basic_setup /init/main.c:887
 [<ffffffff822fd81a>] kernel_init_freeable+0x2bb/0x382 /init/main.c:1008
 [<ffffffff810004a6>] kernel_init+0x16/0x150 /init/main.c:938
 [<ffffffff81dcb53c>] ret_from_fork+0x7c/0xb0 /arch/x86/kernel/entry_64.S:347
DBG: cpu = ffff88041fc95df0

Previous read of size 1 by thread T371 (K368):
 [<ffffffff815b3a37>] acpi_ns_validate_handle+0x39/0x51 /drivers/acpi/acpica/nsutils.c:575
 [<ffffffff815b3fa1>] acpi_evaluate_object+0x51/0x397 /drivers/acpi/acpica/nsxfeval.c:196
 [<ffffffff8158178b>] acpi_evaluate_integer+0x8c/0xd8 /drivers/acpi/utils.c:278
 [<ffffffff815c523d>] acpi_ac_get_state+0x5d/0xa5 /drivers/acpi/ac.c:127
 [<ffffffff815c5356>] get_ac_property+0x31/0x75 /drivers/acpi/ac.c:151
 [<     inlined    >] power_supply_update_leds+0x3c/0x260 power_supply_update_gen_leds /drivers/power/
power_supply_leds.c:122
 [<ffffffff8194c86c>] power_supply_update_leds+0x3c/0x260 /drivers/power/power_supply_leds.c:164
 [<ffffffff8194b8c3>] power_supply_changed_work+0xb3/0x110 /drivers/power/power_supply_core.c:86
 [<ffffffff810aaf3e>] process_one_work+0x2ee/0x730 /kernel/workqueue.c:2027
 [<ffffffff810abe9b>] worker_thread+0xcb/0x740 /kernel/workqueue.c:2160
 [<ffffffff810b5a2b>] kthread+0x14b/0x170 /kernel/kthread.c:207
 [<ffffffff81dcb53c>] ret_from_fork+0x7c/0xb0 /arch/x86/kernel/entry_64.S:347
DBG: cpu = 0

DBG: addr: ffff8801e5911e59
DBG: first offset: 0, second offset: 0
DBG: T1 clock: {T1: 71561485, T371: 17781}
DBG: T371 clock: {T371: 18001}
==================================================================
```

```
==================================================================
ThreadSanitizer: data-race in scsi_schedule_eh

Write of size 4 by thread T6 (K6):
 [<ffffffff817a2dd3>] scsi_schedule_eh+0x63/0xf0 /drivers/scsi/scsi_error.c:84
 [<ffffffff817e23fe>] ata_std_sched_eh+0x8e/0xc0 /drivers/ata/libata-eh.c:1004
 [<ffffffff817e3398>] ata_port_schedule_eh+0x38/0x50 /drivers/ata/libata-eh.c:1044
 [<ffffffff817d8c48>] __ata_port_probe+0xd8/0x100 /drivers/ata/libata-core.c:6115
 [<ffffffff817d8cb0>] ata_port_probe+0x40/0x80 /drivers/ata/libata-core.c:6125
 [<ffffffff817d8d45>] async_port_probe+0x55/0x90 /drivers/ata/libata-core.c:6150
 [<ffffffff810bfe48>] async_run_entry_fn+0x88/0x270 /kernel/async.c:123
 [<ffffffff810aaf3e>] process_one_work+0x2ee/0x730 /kernel/workqueue.c:2027
 [<ffffffff810abe9b>] worker_thread+0xcb/0x740 /kernel/workqueue.c:2160
 [<ffffffff810b5a2b>] kthread+0x14b/0x170 /kernel/kthread.c:207
 [<ffffffff81dcb53c>] ret_from_fork+0x7c/0xb0 /arch/x86/kernel/entry_64.S:347
DBG: cpu = ffff88041fc95df0

Previous read of size 4 by thread T155 (K683):
 [<ffffffff817a41e0>] scsi_error_handler+0x90/0x9f0 /drivers/scsi/scsi_error.c:2157 (discriminator 1)
 [<ffffffff810b5a2b>] kthread+0x14b/0x170 /kernel/kthread.c:207
 [<ffffffff81dcb53c>] ret_from_fork+0x7c/0xb0 /arch/x86/kernel/entry_64.S:347
DBG: cpu = 0

DBG: addr: ffff8800db5168e0
DBG: first offset: 0, second offset: 0
DBG: T6 clock: {T6: 4542335, T155: 9228}
DBG: T155 clock: {T155: 9244}
==================================================================
```

```
==================================================================
ThreadSanitizer: data-race in ida_pre_get

Read of size 4 by thread T1 (K1):
 [<     inlined    >] ida_pre_get+0x2f/0x170 __idr_pre_get /lib/idr.c:195
 [<ffffffff814efdaf>] ida_pre_get+0x2f/0x170 /lib/idr.c:897
 [<ffffffff814f0ae4>] ida_simple_get+0x54/0x110 /lib/idr.c:1094
 [<ffffffff812f1e76>] __kernfs_new_node+0x96/0x150 /fs/kernfs/dir.c:530
 [<ffffffff812f3797>] kernfs_new_node+0x57/0x90 /fs/kernfs/dir.c:558
 [<ffffffff812f5f92>] __kernfs_create_file+0x62/0x140 /fs/kernfs/file.c:920
 [<ffffffff812f6e21>] sysfs_add_file_mode_ns+0x101/0x230 /fs/sysfs/file.c:256
 [<ffffffff812f6fa6>] sysfs_create_file_ns+0x56/0x80 /fs/sysfs/file.c:283
 [<     inlined    >] driver_create_file+0x32/0x50 sysfs_create_file /include/linux/sysfs.h:434
 [<ffffffff817747b2>] driver_create_file+0x32/0x50 /drivers/base/driver.c:107
 [<ffffffff81772bf9>] bus_add_driver+0x219/0x330 /drivers/base/bus.c:698
 [<ffffffff81774967>] driver_register+0xd7/0x1b0 /drivers/base/driver.c:167
 [<ffffffff81776aee>] __platform_driver_register+0x9e/0xb0 /drivers/base/platform.c:566
 [<ffffffff81776ce5>] platform_driver_probe+0x55/0x140 /drivers/base/platform.c:615
 [<ffffffff81777514>] platform_create_bundle+0xc4/0xf0 /drivers/base/platform.c:677
 [<ffffffff8233e9d9>] i8042_init+0x68d/0x6cb /drivers/input/serio/i8042.c:1488
 [<ffffffff8100032d>] do_one_initcall+0xbd/0x220 /init/main.c:794
 [<     inlined    >] kernel_init_freeable+0x2bb/0x382 do_initcall_level /init/main.c:860
 [<     inlined    >] kernel_init_freeable+0x2bb/0x382 do_initcalls /init/main.c:868
 [<     inlined    >] kernel_init_freeable+0x2bb/0x382 do_basic_setup /init/main.c:887
 [<ffffffff822fd81a>] kernel_init_freeable+0x2bb/0x382 /init/main.c:1008
 [<ffffffff810004a6>] kernel_init+0x16/0x150 /init/main.c:938
 [<ffffffff81dcb53c>] ret_from_fork+0x7c/0xb0 /arch/x86/kernel/entry_64.S:347
DBG: cpu = ffff88041fd95df0

Previous write of size 4 by thread T458 (K455):
 [<ffffffff814eec85>] get_from_free_list+0x85/0xc0 /lib/idr.c:75
 [<ffffffff814f0a46>] ida_get_new_above+0x2a6/0x2f0 /lib/idr.c:994
 [<ffffffff814f0b09>] ida_simple_get+0x79/0x110 /lib/idr.c:1098
 [<ffffffff812f1e76>] __kernfs_new_node+0x96/0x150 /fs/kernfs/dir.c:530
 [<ffffffff812f3797>] kernfs_new_node+0x57/0x90 /fs/kernfs/dir.c:558
 [<ffffffff812f5f92>] __kernfs_create_file+0x62/0x140 /fs/kernfs/file.c:920
 [<ffffffff812f6e21>] sysfs_add_file_mode_ns+0x101/0x230 /fs/sysfs/file.c:256
 [<     inlined    >] internal_create_group+0x16d/0x3f0 create_files /fs/sysfs/group.c:58
 [<ffffffff812f804d>] internal_create_group+0x16d/0x3f0 /fs/sysfs/group.c:116
 [<     inlined    >] sysfs_create_groups+0x64/0xc0 sysfs_create_group /fs/sysfs/group.c:138
 [<ffffffff812f8484>] sysfs_create_groups+0x64/0xc0 /fs/sysfs/group.c:165
 [<     inlined    >] device_add+0x6ca/0x9b0 device_add_groups /drivers/base/core.c:459
 [<     inlined    >] device_add+0x6ca/0x9b0 device_add_attrs /drivers/base/core.c:486
 [<ffffffff8176f2fa>] device_add+0x6ca/0x9b0 /drivers/base/core.c:1037
 [<     inlined    >] serio_handle_event+0x258/0x3e0 serio_add_port /drivers/input/serio/serio.c:560
 [<ffffffff8190dca8>] serio_handle_event+0x258/0x3e0 /drivers/input/serio/serio.c:228
 [<ffffffff810aaf3e>] process_one_work+0x2ee/0x730 /kernel/workqueue.c:2027
 [<ffffffff810abe9b>] worker_thread+0xcb/0x740 /kernel/workqueue.c:2160
 [<ffffffff810b5a2b>] kthread+0x14b/0x170 /kernel/kthread.c:207
 [<ffffffff81dcb53c>] ret_from_fork+0x7c/0xb0 /arch/x86/kernel/entry_64.S:347
DBG: cpu = 0

DBG: addr: ffff8801e598866c
DBG: first offset: 0, second offset: 0
DBG: T1 clock: {T1: 76531585, T458: 19098}
DBG: T458 clock: {T458: 39060}
==================================================================
```

```
==================================================================
ThreadSanitizer: data-race in flush_old_exec

Write of size 4 by thread T271 (K800):
 [<     inlined    >] flush_old_exec+0x2ac/0xea0 de_thread /fs/exec.c:994
 [<ffffffff81246cec>] flush_old_exec+0x2ac/0xea0 /fs/exec.c:1066
 [<ffffffff812c3487>] load_elf_binary+0x4d7/0x2940 /fs/binfmt_elf.c:722
 [<ffffffff812466f3>] search_binary_handler+0x143/0x2f0 /fs/exec.c:1378
 [<     inlined    >] do_execve_common.isra.29+0x7f6/0xa20 exec_binprm /fs/exec.c:1415
 [<ffffffff81248496>] do_execve_common.isra.29+0x7f6/0xa20 /fs/exec.c:1512
 [<     inlined    >] SyS_execve+0x35/0x50 do_execve /fs/exec.c:1554
 [<     inlined    >] SyS_execve+0x35/0x50 SYSC_execve /fs/exec.c:1608
 [<ffffffff81248bd5>] SyS_execve+0x35/0x50 /fs/exec.c:1603
 [<ffffffff81dd2ed9>] stub_execve+0x69/0xa0 /arch/x86/kernel/entry_64.S:661
DBG: cpu = ffff88041fc96030

Previous read of size 4 by thread T1 (K1):
 [<     inlined    >] wait_consider_task+0xc7/0x1690 eligible_child /kernel/exit.c:936
 [<ffffffff81082a57>] wait_consider_task+0xc7/0x1690 /kernel/exit.c:1302
 [<     inlined    >] do_wait+0x1ec/0x3e0 do_wait_thread /kernel/exit.c:1419
 [<ffffffff8108420c>] do_wait+0x1ec/0x3e0 /kernel/exit.c:1488
 [<     inlined    >] SyS_wait4+0xca/0x160 SYSC_wait4 /kernel/exit.c:1615
 [<ffffffff8108497a>] SyS_wait4+0xca/0x160 /kernel/exit.c:1584
 [<ffffffff81dd2929>] system_call_fastpath+0x16/0x1b /arch/x86/kernel/entry_64.S:422
DBG: cpu = 0

DBG: addr: ffff88011498f198
DBG: first offset: 0, second offset: 0
DBG: T271 clock: {T271: 14713, T1: 62714630}
DBG: T1 clock: {T1: 62714663}
==================================================================
```

```
==================================================================
ThreadSanitizer: data-race in skb_queue_tail

Write of size 8 by thread T289 (K822):
 [<     inlined    >] skb_queue_tail+0x63/0xc0 __skb_insert /include/linux/skbuff.h:1245
 [<     inlined    >] skb_queue_tail+0x63/0xc0 __skb_queue_before /include/linux/skbuff.h:1351
 [<     inlined    >] skb_queue_tail+0x63/0xc0 __skb_queue_tail /include/linux/skbuff.h:1385
 [<ffffffff81a78963>] skb_queue_tail+0x63/0xc0 /net/core/skbuff.c:2320
 [<ffffffff81adf7ef>] __netlink_sendskb+0x4f/0x90 /net/netlink/af_netlink.c:1748
 [<     inlined    >] netlink_broadcast_filtered+0x2d7/0x540 netlink_broadcast_deliver /net/netlink/af_netlink.c:1943
 [<     inlined    >] netlink_broadcast_filtered+0x2d7/0x540 do_one_broadcast /net/netlink/af_netlink.c:2010
 [<ffffffff81ae18d7>] netlink_broadcast_filtered+0x2d7/0x540 /net/netlink/af_netlink.c:2055
 [<ffffffff814f8926>] kobject_uevent_env+0x4a6/0x6c0 /lib/kobject_uevent.c:317
 [<ffffffff814f8b63>] kobject_uevent+0x23/0x40 /lib/kobject_uevent.c:375
 [<ffffffff81773616>] uevent_store+0x76/0x80 /drivers/base/core.c:419
 [<ffffffff81771986>] dev_attr_store+0x46/0x70 /drivers/base/core.c:136
 [<ffffffff812fb501>] sysfs_kf_write+0xa1/0xd0 /fs/sysfs/file.c:115
 [<ffffffff812fa155>] kernfs_fop_write+0x165/0x210 /fs/kernfs/file.c:304
 [<ffffffff8123c783>] vfs_write+0x113/0x310 /fs/read_write.c:532
 [<     inlined    >] SyS_write+0x6b/0xe0 SYSC_write /fs/read_write.c:583
 [<ffffffff8123d9fb>] SyS_write+0x6b/0xe0 /fs/read_write.c:575
 [<ffffffff81dd2929>] system_call_fastpath+0x16/0x1b /arch/x86/kernel/entry_64.S:422
DBG: cpu = ffff88041fc16030

Previous read of size 8 by thread T288 (K821):
 [<     inlined    >] datagram_poll+0x105/0x1d0 skb_queue_empty /include/linux/skbuff.h:906
 [<ffffffff81a85635>] datagram_poll+0x105/0x1d0 /net/core/datagram.c:864
 [<ffffffff81a6ce4b>] sock_poll+0x9b/0x220 /net/socket.c:1162
 [<     inlined    >] ep_send_events_proc+0x194/0x340 ep_item_poll /fs/eventpoll.c:800
 [<ffffffff812a7e34>] ep_send_events_proc+0x194/0x340 /fs/eventpoll.c:1509
 [<ffffffff812a8c9a>] ep_scan_ready_list+0x12a/0x360 /fs/eventpoll.c:626
 [<     inlined    >] ep_poll+0x19e/0x5e0 ep_send_events /fs/eventpoll.c:1557
 [<ffffffff812a90ce>] ep_poll+0x19e/0x5e0 /fs/eventpoll.c:1660
 [<     inlined    >] SyS_epoll_wait+0x102/0x130 SYSC_epoll_wait /fs/eventpoll.c:1998
 [<ffffffff812ab182>] SyS_epoll_wait+0x102/0x130 /fs/eventpoll.c:1963
 [<ffffffff81dd2929>] system_call_fastpath+0x16/0x1b /arch/x86/kernel/entry_64.S:422
DBG: cpu = 0

DBG: addr: ffff8800dbbf0490
DBG: first offset: 0, second offset: 0
DBG: T289 clock: {T289: 1966177, T288: 238009}
DBG: T288 clock: {T288: 238635}
==================================================================
```