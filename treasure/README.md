Giờ ta sẽ nhìn sơ qua file binary được cung cấp và các chế độ bảo vệ

![image](https://user-images.githubusercontent.com/114044703/213359025-fd77e75a-2f09-4170-aac5-d656d2453d4a.png)


Có vẻ như flag được giấu bên trong chương trình vì khi thực thi chương  trình chỉ in ra phần đầu của flag

Ta sẽ nhìn hàm main thông qua trình ida:

![image](https://user-images.githubusercontent.com/114044703/213359138-d872b932-e66d-41c9-8dab-800ad3cedda3.png)


Hàm main sẽ in ra part2 của flag là ```4_t1ny_tr34sur3```

Nhìn sơ các chuỗi trong file binary(bằng lệnh strings) thì tìm được part 3:

![image](https://user-images.githubusercontent.com/114044703/213359377-c78ecaea-a55d-4038-814a-614a8e764c82.png)

Ta ghép các phần lại thành một flag hoàn chỉnh

```KCSC{4_t1ny_tr34sur3_27651d2df78e1998}```

