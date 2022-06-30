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
