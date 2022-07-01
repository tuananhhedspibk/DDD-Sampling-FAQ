## Chương 4: Repository

### Sử dụng Repository để update một phần của Aggregate liệu có ổn ?

Không nên làm điều này, nguyên nhân là do khi update một phần của entity ta cần rất nhiều các điều kiện if
hoặc các variation khác nên điều này sẽ làm cho nghiệp vụ ở tầng domain bị lộ lọt xuống tầng infrastructure.

Các logic update chỉ nên thực hiện ở tầng domain, còn repository sẽ chỉ ghi đè giá trị vào DB mà thôi.
Điều này sẽ sẽ làm giảm đi sự ràng buộc giữa tầng domain và tầng infra qua đó tăng tính mở rộng cho hệ thống cũng như thực hiện test một cách dễ dàng.

### Có nên viết method lấy về số lượng bản ghi hoặc check bản ghi có tồn tại hay không ở repository ?

Hoàn toàn có thể vì repository là object biểu thị việc tập hợp của:
- Entity
- Value-Object
nên việc viết các method lấy về số lượng bản ghi hoặc kiểm tra sự tồn tại của bản ghi là không có gì mâu thuẫn cả.

### Nên thực thi việc sắp xếp dữ liệu ở repository như thế nào ?

Ta hoàn toàn có thể truyền param dùng cho điều kiện sắp xếp vào repository bằng hình thức enum như sau:

```TS
enum UserOrderKey {
  NAME,
  USER_ID,
};
```

Thế nhưng có một vấn đề đó là OrderKey liệu có nên nằm ở tầng domain.

Nếu OrderKey được biểu thị bởi tên các cột trong DB thì điều đó là không nên thế nhưng nếu nó biểu thị bởi các thuộc tính của entity
thì việc nằm ở tầng Domain hoàn toàn không có vấn đề gì cả. Việc thực thi sắp xếp (câu `ORDER BY` trong SQL) không nên để lộ lọt vào tầng domain.

### Liệu có thể thực thi phân trang ở Repository ?

Nên trả về dưới dạng một object có biểu thị page như sau:

```TS
class Page<T> {
  items: T[];
  paging: Paging;
}

class Paging {
  totalCount: number;
  pageSize: number;
  pageNumber: number;
}
```

Câu request sẽ sử dụng object như sau:

```TS
class PagingCondition {
  pageSize: number;
  pageNumber: number;
}

abstract class UserRepository {
  fetchPageByName(name: string, pagingCondition: PagingCondition): Page<User>;
}
```

Xử lí paging sẽ được thực thi ở tầng infra.

Thế nhưng cũng sẽ xuất phát những câu hỏi cho rằng liệu thông tin về page có nên được định nghĩa ở tầng domain. Định nghĩa trừu tượng của page đó là `là dữ liệu được trả về chứa các thông tin về số lượng các bản ghi liên quan trong một tập hợp`, nên Repository trả về một phần dữ liệu của tập hợp dưới hình thức page là hoàn toàn hợp lệ.

Điều quan trọng hơn cả đó là việc che dấu đi phần triển khai paging nằm ở tầng infra.

### Có thể update nhiều entities cùng một lúc ? 

Hoàn toàn có thể. Ngoài việc update riêng lẻ từng entity một, ta hoàn toàn có thể gom nhóm để update nhiều entities cùng một lúc.

```TS
abstract class UserRepository {
  insert(user: User);
  insert(users: User[]);
}
```

Hàm insert 1 user duy nhất được gọi từ use-case tuy nhiêu nếu được gọi đi gọi lại nhiều lần với 1 use-case sẽ gây ảnh hưởng đến performance của hệ thống
nên bạn có thể cân nhắc việc sử dụng hàm batch-insert.

Tuy nhiên độ phức tạp khi sử dụng hàm batch-insert sẽ tăng lên đôi chút,
số lượng test-case cũng sẽ tăng theo nên nếu không có vấn đề đáng kể với performance thì việc sử dụng batch-insert cũng không quá cần thiết.

### Có thể sử dụng Pessimism Lock ở Repository

Khi đó nếu tầng domain không bị phụ thuộc vào bất kì một công nghệ nào thì điều đó là hoàn toàn có thể.

```TS
abstract class UserRepository {
  findByUserId(userId: UserId, exclusiveAccess: boolean);
}
```

Tham số `exclusiveAccess` không phải là việc lấy một khoá gì đó từ DB mà là một phần cấu trúc của Repository nên việc định nghĩa ở tầng domain là hoàn toàn có thể.

Khi transaction ở use-case bắt đầu thì ở tầng infra sẽ lấy về lock, nhờ đó ở cùng một thời điểm khi cùng truy cập vào cùng một tài nguyên thì xử lí của transaction nếu sẽ được ưu tiên thực thi trước so với xử lí của method `findByUserId`.

Ngoài ra ta cũng có thể tách riêng phần xử lí `exclusive` này thành một method riêng với tên là `findByUserIdExclusively`.

### Liệu có thể triển khai insert và update cùng một lúc ? 

Hoàn toàn được. Tuy nhiên nên cân nhắc giữa ưu, nhược điểm của phương pháp này.
Ưu điểm đó là việc chỉ cần gọi đến một method duy nhất mà không cần phân biệt đâu là `save`, đâu là `update`.
Nhược điểm đó là phần triển khai trong repository sẽ rất phức tạp do việc phân biệt rõ ràng giữa `save` và `update`. Từ đó dẫn đến tính mở rộng, bảo trì của code sẽ bị ảnh hưởng.

Trên thực tế việc chia ra `save`, `update` ngay từ tầng domain cũng không phải là ít.

### Update object con của Aggregate như thế nào ?

Lấy ví dụ với User Entity với mailAddresses là tập hợp các Email của user đó.

```TS
class User {
  name: string;
  mailAddresses: Email[];
}

abstract class UserRepository {
  insert(user: User);
  update(user: User);
}
```

Xét method update, ta truyền vào nó một user instance với nhiều EmailAddresses là object con. Vấn đề bây giờ là sẽ tạo mới MailAddress hay cập nhật hay xoá ?

Cách đơn giản nhất đó là xoá toàn bộ các bản ghi `email_address` gắn liền với user và insert mới toàn bộ các email mà user entity đang có. Cách làm này khó phát sinh bug và rất đơn giản để triển khai, xong vấn đề là nếu như một bảng khác cũng sử dụng khoá ngoại là email address hoặc giá trị `created_at` cần thiết cho xử lí logic thì cách làm này không phát huy được hiệu quả.

Ngoài ra còn một cách khác đó là so sánh các email hiện có trong DB với các email truyền vào, xử lí các điều kiện rẽ nhánh `if` để từ đó đưa ra kết luận cuối cùng đó là `xoá`, `update hay `tạo mới`. Tuy nhiên cách làm này khá phức tạp và cần phải viết test cẩn thận.

### Nên định nghĩa Repository ở tầng nào, liệu có thể định nghĩa ở tầng usecase hay không ?

Nên định nghĩa ở tầng domain. Lí do là vì Repository biểu diễn phạm vi của Aggregate dưới dạng code. Còn nếu định nghĩa ở tầng use-case thì các thông tin liên quan tới Aggregate sẽ không còn nằm trong tầng domain nữa.

Ngoài ra nếu định nghĩa ở tầng use-case thì với một entity có thể cần tạo nhiều repositories khác nhau nên dù phạm vi của Aggregate có khác nhau đi chăng nữa thì tầng domain cũng khó có thể nhận diện được.

Phạm vi của Aggregate là rất quan trọng đối với tầng domain nên Repository biểu thị phạm vi của Aggregate cũng nên được định nghĩa ở tầng domain.

### Với các trường hợp lưu file hoặc lưu dữ liệu ở service bên ngoài liệu có nên gom tất cả lại vào trong một repository ?

Tuỳ vào từng tính chất mà nên tách riêng thì hơn. Đặc biệt là với các trường hợp cần role-back thì việc có thể role-back lại hay không thì tuỳ vào từng use-case. Với các use-case có sử dụng transaction thì ta có thể ngầm hiểu rằng có xử lí ở đâu đó cần phải role-back lại.

Với những trường hợp lưu dữ liệu thì nếu quá trình role-back không rõ ràng sẽ dẫn tới các phát sinh ngoài ý muốn từ đó làm cho dữ liệu không đồng nhất.

Do đó nên đặt tên để có thể dễ phân biệt nhất. VD: với lưu file ta có thể đặt là `XXXStorage` còn với các Repo có tương tác với API ngoài thì ta đặt là `XXXClient`.

Còn với những trường hợp không thể role-back được thì nếu không có sự nhầm lẫn về mặt tính chất, ta có thể đặt chung vào một Repository cũng được.
