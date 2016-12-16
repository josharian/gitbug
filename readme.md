Simple in-memory FUSE filesystem to illustrate a git bug.
Code adapted from https://github.com/bazil/fuse/pull/83.

To reproduce, install OSXFUSE, then:

```bash
$ git version
git version 2.11.0
$ go get -u github.com/bazil/fuse
$ go get -u golang.org/x/net/context
$ git clone https://github.com/josharian/gitbug.git
$ cd gitbug
$ go run main.go ~/memfs

$ # separate shell:
$ git clone https://github.com/josharian/gitbug.git ~/memfs/gitbug
$ # succeeds

$ # kill the memfs process, then:
$ umount ~/memfs
$ go run main.go -crash ~/memfs

$ # separate shell:
$ git clone https://github.com/josharian/gitbug.git ~/memfs/gitbug
Cloning into '/Users/josh/memfs/gitbug'...
Bus error: 10

$ # or under lldb (first kill the go program, umount, and restart it):
$ lldb git
(lldb) target create "git"
Current executable set to 'git' (x86_64).
(lldb) run clone https://github.com/josharian/gitbug.git ~/memfs/gitbug

Process 45559 launched: '/usr/local/bin/git' (x86_64)

Cloning into '/Users/josh/memfs/gitbug'...
Process 45559 stopped
* thread #1: tid = 0xf6c1a, 0x0000000100096f90 git`git_config_set_multivar_in_file_gently + 1293, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=10, address=0x1002d8023)
    frame #0: 0x0000000100096f90 git`git_config_set_multivar_in_file_gently + 1293
git`git_config_set_multivar_in_file_gently:
->  0x100096f90 <+1293>: movzbl -0x1(%rbx,%r14), %ecx
    0x100096f96 <+1299>: cmpl   $0xa, %ecx
    0x100096f99 <+1302>: movl   %eax, %r13d
    0x100096f9c <+1305>: movl   $0x1, %eax
(lldb) bt
* thread #1: tid = 0xf6c1a, 0x0000000100096f90 git`git_config_set_multivar_in_file_gently + 1293, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=10, address=0x1002d8023)
  * frame #0: 0x0000000100096f90 git`git_config_set_multivar_in_file_gently + 1293
    frame #1: 0x0000000100097197 git`git_config_set_multivar_in_file + 18
    frame #2: 0x0000000100037553 git`init_db + 1720
    frame #3: 0x000000010001a6d9 git`cmd_clone + 1565
    frame #4: 0x00000001000026be git`handle_builtin + 529
    frame #5: 0x0000000100002042 git`cmd_main + 259
    frame #6: 0x0000000100077ff8 git`main + 76
    frame #7: 0x00007fffc4f69255 libdyld.dylib`start + 1
(lldb) ^D
```

