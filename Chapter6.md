## Chương 6: Thực thi tầng domain

### Nên đặt tên DomainService như thế nào ?

Không nên đặt tên theo kiểu `XXXService` mà nên đặt tên sát với nghiệp vụ nhất có thể ví dụ như `TaxCalculation`. Không những thế việc đặt tên theo format `XXXService` sẽ khiến cho tên rất dài, ngoài ra do không biết rõ được class sẽ đảm nhận nghiệp vụ gì nên việc thêm mới các methods vào class cũng sẽ rất khó khăn.

Nếu vô tình thêm quá nhiều methods vào cũng sẽ khiến cho class trở nên phình to hơn bình thường.

> Hãy xem xét kĩ việc đặt tên dựa theo chức năng, nghiệp vụ mà service đảm nhiệm

### Có nên sử dụng Repository ở DomainService hay không ?

Cùng xem xét từ vai trò của `DomainService`, về cơ bản `DomainService` là nơi định nghĩa các rule hoặc nghiệp vụ mà khi cho vào Entity hay Value-Object trông sẽ rất là kì.

Ta lấy ví dụ với `Booking Entity`, "Khoảng thời gian này đã được đặt lịch từ trước" - nếu `Booking Entity` biết điều này thì sẽ rất là kì. Hơn nữa việc xem xét xem `Booking` có tồn tại hay không, ta cần sử dụng đến Repository bên trong DomainService. Nên ta hoàn toàn có thể sử dụng `Repository` bên trong `DomainService`.

Tuy nhiên cũng cần chú ý đảm bảo việc DomainService không bị quá phình to với nhiều nhiệm vụ đi kèm.
