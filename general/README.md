# Encoding Challenge
Mô tả: đại loại server sẽ tạo 100 test để chúng ta decode, tuy nhiên nếu decode bằng tay sẽ rất mất sức nên ở đây ta sẽ dùng tool của pwntool  
chức năng chính khi dùng pwntool trong bài này là tự động hoá thay vì phải nhập tay  
Có các loại mà chương trình muốn ta decode: hex, base64, utf-8 (dec to ascii), biginit (base 10 to ascii, bài Bytes and Big int trong cryptohack), rot13.  
đầu tiên để decode hex, ta sử dụng câu lệnh sau:  
```python3
res = bytes.fromhex(code).decode()
```
  hoặc có thể là
  ```python3
  res = bytes.fromhex(code) #res ở dòng này là byte (1)
  res = res.decode()        #decode nó thành string (2)
  ```
  cả 2 đều tương tự nhau. Sở dĩ phải decode() nó thành string vì nếu kết thúc dòng 1 mà gửi dữ liệu hay in ra ta sẽ thấy nó có dạng ``` b'abc' ``` và nếu gửi dữ liệu đi thì ta đã nhầm gửi ```b'abc'``` chứ không phải ```abc```
  ___
  tiếp đến để decode base64, ta sử dụnng lệnh sau:
  ```python3
  res = base64.b64decode(code).decode()
  ```
  ở đây ta lưu ý phân biệt giữa b64encode và b64decode, b64encode thì phải đi từ hex, còn b64decode đi từ b64encode, nghĩa là nếu nó cho một đoạn bắt đổi về base64 thì ```đề >>> hex >>> b64encode``` còn cho ta đoạn code base64 thì ```đề >>> b64decode```.
  Bài này người ta bắt decode về base64  
  ___
  tiếp đến để decode rot13, ta dùng câu lệnh sau:
  ```python3
  import codecs
  res = codecs.encode(code, 'rot_13')
  ```
  ở đây tôi tạm hiểu là người ta cho mã đã lùi về sau 13 kí tự (decode), việc của mình là dời lên trước 13 kí tự (encode), đây là tôi thử để xem flag có nghĩa hay không, chứ không hẳn có thể nhìn vào là biết được, nên là phải đoán.
  ![image](https://user-images.githubusercontent.com/111769169/220428419-ec87ca89-f183-492f-aac9-3959208096e3.png)
  ___
  tiếp đến để decode bigint, ta dùng câu lệnh sau:
  ```python3
  from Crypto.Util.number import *
  res = long_to_bytes(int(code, 16)).decode()
  ```
  đây là bài [Bytes and Big Integers](https://cryptohack.org/challenges/general/), hiểu đơn giản là đề cho một số long int rất lớn, và ta chuyển nó thành hex dạng 0x123456789012... ở đây cứ 2 byte hex thì được 1 kí tự ascii, 0x12, 0x34, 0x45, 0x67... Vậy tóm lại ``` long int (đề) >>> hex >>> ascii ``` và đó là lý do tại sao có ```int(code, 16) # đổi long int thành hex ``` và đương nhiên do long_to_bytes() là hàm trả về kiểu bytes, nên ta cần decode để về thành str.
  ___
  tiếp đến để decode utf-8, ta dùng câu lệnh sau:
  ```python3
  res = ""
        for i in range(0, len(code)):
            res += chr(code[i])
  ```
  Tại sao ta phải có ```res = ""```, hiểu đơn giản là ta khai báo biến vậy ``` int a; char b ``` thì khi dùng biến res trong hàm for thì nó hiểu là đang sài res bên ngoài và lưu kết quả lại vào hàm res. Ở đây tôi khai báo là con trỏ không có j nên ```""``` để khi nó ``` += ``` thì nó sẽ không làm thay đổi dữ liệu ta cần
  ___
  *Để chạy được script này thì phải xoá hết các kí tự là có dấu =)))*
  [script tham khảo khác](https://github.com/s-nikravesh/crypto-hack/blob/master/General/Encoding%20Challenge.py)
  ```python3
from pwn import *  # pip install pwntools
from base64 import *
from Crypto.Util.number import *
import codecs
import json

r = remote('socket.cryptohack.org', 13377, level = 'debug')    #tham số level = 'debug' là chạy chương trình với debug để thấy được dữ liệu vào ra rõ ràng   


def json_recv():                          #json nhận dữ liệu      
    line = r.recvline()
    return json.loads(line.decode())


def json_send(hsh):                        #json gửi dữ liệu
    request = json.dumps(hsh).encode()
    r.sendline(request)

for i in range(100):                        #100 test
    received = json_recv()                  #gọi hàm json_recv() để nhận dữ liệu
    print("Received type: ")                
    type = received["type"]                 #nghĩa là trong json nó như z "type" : abc
    print(type)                             #nhận gán biến type = abc
    print("Received encoded value: ")
    code = received["encoded"]
    print(code)
                                            #các if sau là để so sánh
    if type == 'base64':
        res = b64decode(code)
        res = res.decode()
        to_send = {
            "decoded": res
        }

    elif type == 'hex':
        res = bytes.fromhex(code)
        res = res.decode()
        to_send = {
            "decoded": res
        }

    elif type == 'rot13':
        res = codecs.encode(code, 'rot_13')
        to_send = {
            "decoded": res
        }

    elif type == 'utf-8':
        res = ""
        for i in range(0, len(code)):
            res += chr(code[i])
        to_send = {
            "decoded": res
        }
    elif type == 'bigint':
        res = long_to_bytes(int(code, 16)).decode()
        to_send = {
            "decoded": res
        }
    json_send(                  #gọi hàm json_send để gửi
        {"decoded": res}
    )
flag = r.readline()             #nhận 1 dòng in ra từ remote nên mới có r.readline()
print(flag)                     #in ra


  ```
