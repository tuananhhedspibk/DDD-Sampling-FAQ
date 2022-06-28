## Chương 1: Tổng quan về DDD

### Phạm vi ứng dụng của DDD

`Domain`: là cách ứng dụng phần mềm để giải quyết một vấn đề nào đó.
`Domain Model`: là một cách trừu tượng hoá các đối tượng nhằm giải quyết một bài toán (domain) nào đó.

Một phần mềm làm ra cần phải thoả mãn 2 yêu cầu sau:
- Cung cấp một tính năng, dịch vụ nào đó cho người dùng (1)
- Dễ dàng mở rộng, chỉnh sửa (2)

(1): `Domain Model` cần phản ánh đúng nghiệp vụ, vấn đề cần giải quyết.
(2): Áp dụng pattern sử dụng Entity, Repository để đáp ứng được việc thường xuyên thay đổi model.

### Nguyên tắc khi thiết kế

Nhằm tăng tính mở rộng của code, khi thiết kế cần chú ý 2 điểm sau:
1. Hiểu được nhiệm vụ của từng class, các thành viên của class (data, method) cần gắn kết chặt chẽ với nhau. Giảm tính phụ thuộc lẫn nhau giữa các classes.
2. Có thể test hệ thống một cách dễ dàng.

### Thực thi DDD với hệ thống sẵn có

Có 2 cách tiếp cận:
- Bottom-Up: đi theo từng bước một, refactor từng entity, value-object. **Ưu điểm** ở đây đó là: chi phí thấp, hệ thống luôn trong trại thái có thể sử dụng được. **Nhược điểm** là có thể sẽ không cải thiện được vấn đề gì cả.
- Top-Down: Viết lại nghiệp vụ cho từng layer một. **Ưu điểm**: do đã quyết định sẵn về cách cải thiện hệ thống nên hiệu quả sẽ cao hơn. **Nhược điểm** là việc kết hợp kiến trúc mới với kiến trúc hiện tại cần có sự đồng ý của team cũng như tốn khá nhiều chi phí để thực hiện. Nên về cơ bản cách tiếp cận này khó hơn so với cách tiếp cận **Bottom-Up**

### Nên bắt đầu từ đâu ?

Có 2 cách tiếp cận:
- Domain modeling: Với những thứ được làm mới, ngay từ đầu cần có sự hiểu rõ vấn đề, đối sách phù hợp cũng như có được phương án cải thiện những cái đã có sẵn.
- Làm theo các pattern: chú trọng vào tính mở rộng của hệ thống, không quan trọng làm mới hay cải thiện những cái đã có

### Với những ý tưởng mới, việc áp dụng DDD liệu có phù hợp ?

Điều quan trọng ở đây không phải là ý tưởng - nghiệp vụ mới hay cũ mà là việc chúng ta hiểu rằng mình phải giải quyết bài toán gì.

### DDD - hợp và không hợp

> DDD rất thích hợp cho việc xử lí các domain có tính phức tạp cao

Tuy nhiên với các ứng dụng đơn thuần CRUD thì việc áp dụng DDD không hẳn đã là hợp lí vì những lí do sau:
- Chi phí để hiểu DDD
- Kể cả khi hiểu về DDD thì việc áp dụng cũng gặp rất nhiều khó khăn
