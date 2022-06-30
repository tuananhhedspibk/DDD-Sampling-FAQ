## Chương 2: Modeling

Trong DDD ngay từ đầu Model không thể hoàn chỉnh ngay được mà cần phải có sự đổi mới, cập nhật thường xuyên.

### Ví dụ về modeling

Lấy ví dụ về một hệ thống quản lí việc tuyển dụng cũng như các ứng viên ứng tuyển của một công ty.

Dưới đây là những thứ cần xem xét:
1. Sơ đồ hệ thống
2. Sơ đồ usecase
3. Sơ đồ domain model
4. Sơ đồ object

### Sơ đồ hệ thống

Biểu thị:
- Hệ thống sẽ phát triển
- Các actors liên quan
- Các hệ thống liên kết bên ngoài

![Screen Shot 2022-02-06 at 18 08 19](https://user-images.githubusercontent.com/15076665/152674332-0a1c3a08-1015-4622-a93c-2848161789aa.png)

Tuy đơn giản nhưng ta vẫn cần đặt ra những câu hỏi sau nếu muốn điều chỉnh hệ thống:
- Ai sẽ là người sử dụng hệ thống ?
- Việc ứng tuyển sẽ được thực hiện trực tiếp đối với hệ thống ?

Quá trình trả lời các câu hỏi trên sẽ diễn ra khi tiến hành modeling, kết quả thảo luận sẽ được thể hiện thông qua sơ đồ domain model.

Hệ thống lần này sẽ chỉ do người phụ trách việc tuyển dụng sử dụng. Việc ứng tuyển sẽ thông qua form, dữ liệu thu thập từ form sẽ được nhập bằng tay vào hệ thống.

Tuy nhiên nếu sau này muốn phát triển hệ thống theo hướng có thể ứng tuyển trực tiếp thì phạm vi của hệ thống sẽ được mở rộng lên rất nhiều. Vậy nên hãy quyết định vấn đề này ngay từ đầu để có thể biểu thị nó một cách chính xác nhất trên sơ đồ hệ thống.

### Sơ đồ usecase

Sơ đồ này là chi tiết của hệ thống đối với từng actor (tác tử) có trong hệ thống.

![Screen Shot 2022-02-23 at 16 42 26](https://user-images.githubusercontent.com/15076665/155278529-cbde8a2a-5af2-40df-8341-e700ea4b2324.png)

Hơn nữa trong quá trình thiết kế sơ đồ usecase, nếu phát sinh vấn đề gì, ta hoàn toàn có thể chỉnh sửa lại sơ đồ thiết kế hệ thống.

### Sơ đồ DomainModel / Sơ đồ Object

2 sơ đồ này có mối liên hệ trừu tượng / cụ thể.

Sơ đồ DomainModel là sơ đồ đơn giản hoá từ sơ đồ class, cần tuân theo những quy định sau:
- Cần phải ghi những thuộc tính tiêu biểu của object, có thể không cần phải ghi method.
- Cần phải ghi rõ ràng các nghiệp vụ.
- Biểu thị rõ tính liên quan giữa các objects.
- Định nghĩa độ đa dạng
- Định nghĩa phạm vi của Aggregate
- Dịch nghĩa rõ ràng các thuật ngữ của nghiệp vụ.

Sơ đồ object là sơ đồ đưa ra ví dụ cụ thể của DomainModel. Cần có sự tham gia của người xây dựng mô hình.

Ví dụ:
Dưới đây là sơ đồ object khi sơ đồ usecase ở phần trước được triển khai. Gồm 2 trường hợp:
- Trượt phỏng vấn server engineer từ vòng 1
- Đang chuẩn bị phỏng vấn vòng 2

<img width="770" alt="Screen Shot 2022-02-26 at 13 16 22" src="https://user-images.githubusercontent.com/15076665/155828540-8d37ad97-0e44-48c5-9aef-d0673e88254a.png">

Từ sơ đồ object trên, ta có thể trừu tượng hoá để có sơ đồ Domain Model như sau:

<img width="832" alt="Screen Shot 2022-02-26 at 13 20 25" src="https://user-images.githubusercontent.com/15076665/155828684-86b234cf-8511-42a2-a3ef-0ae6ab1d4a45.png">

Sơ đồ DomainModel luôn phải có
- Entity, Value-Object
- Các nghiệp vụ đi kèm

Ví dụ như: [Trạng Thái Tuyển Dụng] sẽ bao gồm những giá trị gì ? -> Đây chính là yếu tố cần phải cân nhắc.

Tuy nhiên khi tiến hành mô hình hoá có thể sẽ có những yếu tố chưa thể quyết định ngay được, hãy ghi chú lại trong bản thiết kế cho dù điều này sẽ khiến bản thiết kế của bạn trông không được chuẩn chỉnh cho lắm.

(4) chính là độ đa dạng - Liệu giữa `Kết quả tuyển dụng` và `Vị trí tuyển dụng` có cần một mối liên hệ nào đó hay không ? Đây là điều cần phải quyết định

(5) Quyết định phạm vi của Aggregate: với 1 Repository sẽ có 1 Aggregate tương ứng. Ta cần cân nhắc những:
- Entity
- Value-Object
- Repository

nào sẽ được thực thi bên trong một Aggregate. Tuy vậy trong thực tế sẽ có những Aggregate nếu không thực thi trong thực tế sẽ không thấy rõ được tính hiệu quả của chúng. Vậy nên hãy quyết định Aggregate sớm nhất, thực thi nó sớm nhất và chỉnh sửa nó nếu cần thiết.

Trong sơ đồ DomainModel, tham chiếu bên trong Aggregate sẽ được coi là Composition, còn tham chiếu ra bên ngoài sẽ được biểu thị bởi dấu mũi tên như ở sơ đồ trên.

Hãy quyết định các thuật ngữ trong sơ đồ bằng tiếng Anh nhanh nhất có thể để từ đó làm cơ sở trong thiết kế:
- Endpoint URL
- Table Name
- ...

**Tầm quan trọng của sơ đồ object**

Với những người mới bắt đầu (khi chưa có kiến thức về nghiệp vụ) thì việc đọc hiểu sơ đồ `DomainModel` là một điều rất khó khăn. Cách tiếp cận đi từ `Cụ thể` -> `Trừu tượng` sẽ dễ dàng hơn là đi từ `Trừu tượng` -> `Cụ thể`. Vậy nên sơ đồ object sẽ rất quan trọng trong những tình huống như vậy.

**Trình tự thiết kế sơ đồ object và sơ đồ domain model**

Lý thuyết sẽ nói rằng chúng sẽ được thiết kế cùng một lúc nhưng trong thực tế thì sẽ đi từ `Cụ thể` -> `Trừu tượng` từng bước một.

### Sự gắn kết giữa việc triển khai và Modeling

Sau khi hoàn thành sơ đồ DomainModel sẽ là quá trình triển khai. Trong quá trình dev, sẽ phát sinh nhiều vấn đề mà ta cần phải cập nhật DomainModel cũng như cập nhật source code.

Tuy nhiên khi model thay đổi thì việc cập nhật lại source code không hẳn đã đơn giản vì cần phải có sự cân nhắc về việc thay đổi phần nào sao cho phù hợp nhất.

Do đó cần phải làm cho Model và Code gần nhau nhất có thể. Để có thể làm được điều đó ta có thể áp dụng Best Practice đó là `Entity và Value-Object` pattern.

Tuy nhiên khi thay đổi code, ta cần phải thực thi Regression Test bằng tay một cách thường xuyên, dẫn đến việc nếu thực thi test không đầy đủ có thể xảy ra trường hợp code không phản ánh đúng nghiệp vụ hoặc phát sinh bug. Để tránh điều đó xảy ra với DDD thì việc thực thi test tự động là một điều vô cùng quan trọng.

### Sample Code

Với các Factory class, ta nên để các method `create`, `reconstruct` dưới dạng **static method**. Factory class sẽ làm nhiệm vụ tạo các entity instance dựa trên dữ liệu lấy ra từ DB.

Với Value-Object ta có thể triển khai dưới dạng enum như sau:

```TS
enum ScreeningStatus {
  IN_PROGRESS = 'IN_PROGRESS',
  ADOPTED = 'ADOPTED',
  REJECTED = 'REJECTED',
}
```

hoặc dạng class như sau:

```TS
class ScreeningId {
  value: string;

  constructor(value: string) {
    this.value = value;
  }
}
```

### Modeling - Liệu có cần thiết kế toàn bộ 4 loại sơ đồ Model hay không ?

Thực tế là không vì chỉ cần tạo những model cần thiết mà thôi.

4 loại sơ đồ ở đây đó là:
- **Sơ đồ hệ thống**: có thể lược bỏ nếu không liên kết với các hệ thống bên ngoài quá nhiều
- **Sơ đồ Use-Case**: cần thiết khi phát triển các tính năng mới. Tuy nhiên nếu không hiểu rõ được Actor là ai hoặc tính năng này có thể làm được những gì thì việc chỉnh sửa sẽ phát sinh. Nhưng nếu có thể biểu thị được các tính năng hoặc actor thông qua user story ta hoàn toàn có thể bỏ qua sơ đồ này và sử dụng một cách tiếp cận khác.
- **Sơ đồ DomainModel**: cần thiết
- **Sơ đồ Object**: cần thiết

Hai sơ đồ DomainModel và Object thực sự cần thiết là vì kể cả khi tính năng là rất nhỏ nhưng nếu thử phác thảo các sơ đồ Model sẽ có các vấn đề phát sinh khác cần phải xem xét.

### Modeling được thực hiện vào lúc nào ?

Sau khi thiết kế yêu cầu hệ thống và ở các phase dev. Nếu có vấn đề cần phải chỉnh sửa thì nhất thiết ta phải cập nhật các sơ đồ Model.

Trong thực tế sẽ không có phase Modeling mà quá trình này sẽ được tiến hành thông suốt trong toàn bộ quãng thời gian phát triển phần mềm.

### Có cần phải bảo trì các sơ đồ Model hay không ?

`Sơ đồ hệ thống` giúp ta có cái nhìn bao quát nhất về hệ thống nên việc bảo trì sơ đồ này là rất cần thiết.
`Sơ đồ use-case` không cần thiết phải bảo lưu, bảo trì vì khi dev, sơ đồ này sẽ hay thay đổi.
`Sơ đồ object` và `Sơ đồ DomainModel` đều cần phải được bảo trì, cập nhật thường xuyên do ta phải đảm bảo code không quá khác xa so với sơ đồ DomainModel, vậy nên cần tuân theo trình tự `Cập nhật sơ đồ DomainModel` -> `Chỉnh sửa code`

### Liệu có cần các sơ đồ khác ngoài 4 loại sơ đồ trên hay không ?

Không cần thiết. Nếu phát sinh những vấn đề phức tạp như:
- Chuyển trạng thái
- Liên kết với hệ thống ngoài

thì việc tạo thêm các sơ đồ khác là cần thiết.
