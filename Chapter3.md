## Chương 3: Entity và Value-Object

### Phân biệt giữa Entity / Value-Object với Data Model / ORM Class

Entity và Value-Object (VO) là các class được sử dụng để biểu thị DomainModel. Trong quá trình triển khai, ta nên tránh việc các yếu tố như RDB Table hoặc ORM ảnh hưởng tới Entity và VO.

Ngược lại thì Data Model dùng để lưu trữ dữ liệu vào DB, với RDB sẽ được biểu thị thông qua các Table. ORM Class là các class dùng để Mapping giữa Data của chương trình với Database.

Xét ví dụ với Domain Model như sau:

![Screen Shot 2022-02-26 at 17 43 06](https://user-images.githubusercontent.com/15076665/155836692-b88628d7-9f63-41ec-8234-f31f932321e5.png)

Nếu sử dụng Entity / VO ta sẽ có source code như sau:

```TS
class User {
  userId: String;
  name: String;
  mailAddresses: MailAddress[];
}

class MailAddress {
  value: string;

  constructor() {
    // ...
  }
}
```

Với DataModel ta sẽ có như sau:

![Screen Shot 2022-02-26 at 17 43 06](https://user-images.githubusercontent.com/15076665/155837037-2e771526-8218-457b-83cd-9bd194c2ae68.png)


```TS
class User {
  userId: string;
  name: string;
  mailAddresses: UserMailAddress[];
}

class UserMailAddress {
  user: User;
  mailAddress: string;
}
```

User và UserAddress có quan hệ thông qua `userId`. `userId` là khoá ngoại được kéo từ bảng User.
Ta thấy rằng với Data Model ta cần phải truyền vào UserMailAddress một user instance. Điều này sẽ gây ra khó khăn cho quá trình viết unit test vì ngoài việc test `UserMailAddress` ta còn phải quan tâm đến mối liên hệ giữa `UserMailAddress` và `User`. Với Enttiy / VO ta chỉ cần quan tâm đến `User` là đủ.

### Cách sử dụng ORM Class

Như phần trước đã nói, `ORM Class` chỉ nên được sử dụng trong tầng `infrastructure` chứ không nên được sử dụng ở tầng `domain` hoặc tầng `usecase` ngoài ra cũng không nên sử dụng `ORM Class` như là `Entity` hoặc `VO`. 

Mà nên sử dụng bằng cách truyền các entity hoặc VO vào trong một `ORM Class` dưới dạng tham số truyền vào các hàm `insert` hoặc `update` của Repository.

Còn với các method liên quan đến tham chiếu, ta sẽ sử dụng `ORM Class` để lấy dữ liệu từ DB rồi tạo các instance của entity hoặc VO dựa theo kết quả từ DB.

### Tạo Entity

- Với dữ liệu lấy ra từ DB, ta cần những `Factory Methods` chuyên dụng để tái tạo lại entity đúng theo như yêu cầu của nghiệp vụ dựa trên dữ liệu lấy ra được.
- Với những ngôn ngữ chỉ sử dụng được duy nhất một `Primary Constructor` trong khi yêu cầu cần có nhiều constructor thì ta có thể làm như sau:
1. Tạo một private constructor
2. Các factory methods tiếp theo (vd: `create`, `reconstruct`, ...) sẽ gọi đến private constructor đó để tạo instance

```TS
class Task {
  private constructor() {}

  static create() {
    const task = new Task();
    // ...
  }

  static reconstruct() {
    const task = new Task();
    // ...
  }
}
```

Vì ở đây ta định nghĩa `private constructor` nên các class bên ngoài không thể gọi đến constructor của class Task được. Nếu biến constructor trở thành public thì có thể dẫn đến việc khởi tạo các instance với giá trị sai lệch đi so với nghiệp vụ.

```TS
const wrongTask = new Task({ name: "wrong", status: TaskStatus.DONE });
```

`reconstruct` method được sử dụng trong class triển khai các repository để tái cấu trúc lại entity dựa theo dữ liệu lấy ra từ DB.

### Sự khác biệt giữa constructor và factory method

Về bản chất ở đây không có sự khác nhau nhiều nhưng về mặc method name thì constructor method chỉ có thể có tên là `constructor` hoặc cùng tên với class như ở một số ngôn ngữ như Java. Còn với factory method ta có thể để tên là `create` hoặc `reconstruct`.

### Liệu có thể truyền Value-Object vào creation method ?

Hoàn toàn được, tuy nhiên nếu truyền Value-Object vào creation method thì code sẽ dễ đọc hơn một chút. Cùng so sánh 2 pattern dưới đây

```TS
class MailAddress {
  constructor(value: string) {}
}

class User {
  createFromVO(mail: MailAddress) {
    const user = new User(mail); // Pattern-1
  }

  createFromPrimitive(mail: string) {
    const user = new User(new MailAddress(mail)); // Pattern-2
  }
}
```

Như thấy ở pattern-1 ta truyền vào đó MailAddress Value-Object nên sẽ không có trường hợp truyền các string không liên quan đến email vào cả, qua đó có thể nói rằng code sẽ dễ đọc hơn đôi chút.

Ngoài ra bắt buộc phải có sự kiểm tra tính chính xác của email truyền vào. Nếu điều này được thực thi trong `User class` thì sẽ giảm đi tính bảo trì của class cũng như sẽ gây sự khó hiểu cho người đọc code. Hơn nữa nếu class khác cũng sử dụng email thì logic kiểm tra email sẽ không thể tái sử dụng lại được. Do đó quá trình kiểm tra email nên được tiến hành trong `MailAddress class` để vừa đảm bảo tính tái sử dụng code cũng như đảm bảo được nghiệp vụ kiểm tra email.

### Các giá trị được lưu trong entity

**Sử dụng các cột được tạo tự động trong ORM** -  với các thuộc tính `created_at` hoặc `updated_at` thường được tạo tự động bở ORM thì sử dụng chúng ở màn hình liệu có được không ?

Nếu trong trường hợp có hiển thị trên màn hình thì nên định nghĩa các thuộc tính này trong các class Entity hoặc VO và sử dụng các setter method được định nghĩa trong các class đó. Có 2 lí do cho điều đó:

1. `Thời điểm thiết lập giá trị cho chúng` - Vì từ khi entity được tạo cho đến khi truyền vào Repository (do ORM chỉ có thể được sử dụng trong Repository mà thôi) thì các giá trị này sẽ không được thiết lập dẫn đến việc tồn tại các entity instance với `created_at` và `updated_at` = `null` . Đây là điều không nên có.

2. `Giá trị được thiết lập` - Do thời gian được chuyển xuống dưới ORM và thời gian logic ở tầng domain diễn ra là hoàn toàn khác nhau cũng như phía ORM không thể biết chính xác thời điểm logic ở tầng domain diễn ra nên nếu muốn chính xác thời gian diễn ra logic thì nên thiết lập giá trị `created_at` và `updated_at` tại tầng domain.

### Thông tin về version - Optimism lock được sử dụng như thế nào ?

`Optimism lock` thường được sử dụng để tránh tình trạng nhiều user cùng thay đổi tài nguyên cùng một lúc. Tuy nhiên việc thiết lập giá trị này (version) tại tầng domain thì lại vi phạm về quy tắc nghiệp vụ tại tầng domain.

Với thông tin về version, nó sẽ được điều khiển tại tầng infrastructure nên nếu thực thi hoặc điều khiển tại tầng domain thì sẽ là một sự vi phạm về quy tắc của layer. Tuy nhiên ta cũng có thể tư duy theo hướng `Giảm đi tối đa sự vi phạm này` do khi dùng Optimism Lock thì thông tin về version là rất cần thiết.

Có rất nhiều cách triển khai, sau đây là một cách trong số đó. Ta định nghĩa một `OptimismLockable interface` - interface  này sẽ có thuộc tính là `version`. Đoạn code dưới đây sẽ triển khai `OptimismLockable interface` và `Version class`.

```TS
interface OptimismLockable {
  version: Version;
}

class Version {
  constructor () {
    return Version(0);   
  }  
}
```

class và interface trên sẽ được định nghĩa ở `tầng domain` tuy nhiên việc sử dụng nó hay không lại cần có sự cân nhắc nên có thể nói đây là cách giảm thiểu tối đa việc vi phạm quy tắc nghiệp vụ về layer. Entity class sử dụng Optimism Lock sẽ như sau

```TS
class Dog implements OptimismLockable {
  dogId: DogId;
  name: string;
  version: Version; // (1)

  create(name: string) {
    return new Dog(DogId(), name, Version.initial());
  }

  reconstruct(
    dogId: DogId,
    name: string,
    version: Version
  ) {
    return new Dog(dogId, name, version);
  }
}
```

Việc cập nhận version sẽ được triển khai ở tầng infrastructure. Câu SQL để update dữ liệu sẽ như sau:

```SQL
UPDATE Dog
SET name = 'new_name', version = version + 1
WHERE version = 1 AND dogId = 1;
```

Câu update trên sẽ chỉ update những record có version = 1, vậy nên dù cùng được gọi vào 1 thời điểm nhưng sau khi version được increment thì số lượng các record thoả điều kiện version = 1 sẽ là 0. Xử lí sẽ thất bại và ném ra ngoại lệ.

Ngoài ra ta cũng có thể triển khai version ở abstract class (base class của các entities) nhưng nếu làm như vậy thì mọi entities sẽ phải xử lí phần thông tin version nên việc triển khai Optimism Lock thông qua interface vẫn hợp lí hơn cả.

### Khi lấy dữ liệu từ Repository thì có nhất thiết phải kéo về cả những dữ liệu thừa ?

Pattern của DDD ở đây đó là sự đánh đổi hiệu suất khi truy xuất DB để có được tính bảo trì cao cho hệ thống. 

Hãy thử lấy một ví dụ: khi cần update chỉ 2 cột trong DB, điều đó đồng nghĩa với việc ở các tầng domain, infrastructure ta cần các điều kiện `if` cũng như các xử lí `validation` khiến cho xử lí ở các tầng đó trở nên phức tạp và sẽ khó tiến hành mock các hàm ở tầng infrastructure cho mục đích thực hiện unit test.

Để tránh điều đó các method của entity / VO sẽ chỉ tập trung thực hiện domain logic, còn ở tầng infrastructure sẽ chỉ thực thi các truy xuất / update SQL đơn giản để từ đó có thể tăng tính tập trung logic theo từng tầng, giảm đi tính liên kết giữa các tầng cũng như dễ dàng thực thi unit test hơn.

Và do đó ở các repository ta sẽ kéo về toàn bộ các giá trị của các thuộc tính thuộc về entity / VO.

> Điều cần cân nhắc ở đây đó là: HIỆU SUẤT DB vs TÍNH LIÊN TỤC VÀ DỄ TEST

Ngoài ra nếu không chấp nhận việc hiệu suất DB giảm với các xử lí truy vấn dữ liệu ta có thể tách ra thành hai loại model:
- Domain Model: dùng khi cần UPDATE/ INSERT/ DELETE dữ liệu
- Query Model: dùng khi truy vấn dữ liệu.

Với các xử lí batch (cập nhật một lượng lớn dữ liệu) ta có thể cân nhắc việc tăng hiệu suất xử lí và tính bảo trì của hệ thống. Khi đó ta có thể thực thi câu UPDATE QUERY cho nhiều bản ghi dữ liệu cùng một lúc.

### Sử dụng ID

Với auto increment ID ta cũng có thế áp dụng với DDD tuy nhiên có vài điều sau ta cần cân nhắc kĩ lưỡng:

Điều thứ nhất: khi sử dụng auto increment ID ta có thể nghĩ tới 2 cách xử lí sau đây:
1. Ở trong Repository ta sẽ tiến hành tạo ID rồi lưu vào DB
2. Thực hiện quá trình tạo ID từ DB, kéo ID từ DB về Repository rồi xử lí

Với cách xử lí thứ nhất ta cần phải check xem ID truyền vào có null hay không qua đó sẽ phát sinh xử lí `if` không cần thiết từ đó làm giảm đi tính bảo trì của code như ví dụ dưới đây

```TS
const Task = new Task({ content: "Test", userId });

// Ở đây cần check userId có null hay không trước khi truyền vào Task constructor
```

Do đó cách xử lí thứ 2 (đưa auto increment vào DB) là hợp lí hơn.
Ngoài việc sử dụng auto increment ID của DB ta cũng có thể sử dụng `UUID` hoặc `ULID` qua đó làm giảm đi sự phụ thuộc vào DB tăng tính bảo trì cho code.

### Sử dụng Date cho ID liệu có ổn ?

Nếu đảm bảo tính duy nhất của Date thì hoàn toàn OK.
