## Chương 7: Test

### Đối tượng test

Xét các sơ đồ usecase, domain-model, object như dưới đây.

Đầu tiên là sơ đồ use-case

<img width="482" alt="Screen Shot 2022-03-27 at 16 53 52" src="https://user-images.githubusercontent.com/15076665/160272332-352008e0-13a3-490c-a477-d3f2f5bb781b.png">

User có các use-case như:
- Tạo task
- Trì hoãn task
- Hoàn thành task
- Thay đổi người thực hiện task

Sơ đồ Domain-Model, Object sẽ như sau:

<img width="684" alt="Screen Shot 2022-03-27 at 16 54 47" src="https://user-images.githubusercontent.com/15076665/160272362-b81b8098-1481-45c6-839b-7e52bb794754.png">

Chúng ta sẽ thực hiện test với tất cả các tầng:
- Domain
- Usecase
- Infrastructure

### Test Entity

Entity được tạo ra từ những methods dưới đây:
1. Create method (constructor, factory method)
2. Mutation method
3. Reconstruct method

Do đó phía trên là 3 methods ta cần phải test khi tiến hành test entity.

### [Test] Create Method

```TS
class Task {
  id: TaskId;
  name: TaskName;
  userId: UserId;
  status: TaskStatus;
  postponeCount: number;
  dueDate: Date;

  private constructor(id: TaskId, name: TaskName, userId: UserId, status: TaskStatus, postponeCount: number, dueDate: Date) {
    this.id = id;
    this.name = name;
    this.userId = userId;
    this.status = status;
    this.postponeCount = postponeCount;
    this.dueDate = dueDate;
  }

  create(name: TaskName, dueDate: Date, userId: UserId) {
    return new Task(
      TaskId(), name, userId, TaskStatus.UNDONE, 0, dueDate,
    );
  }
}
```

Phía trên ta để `constructor` là `private` qua đó chỉ có trong class mới gọi được đến `constructor` mà thôi, ngoài ra ta chỉ public `create` method mà thôi.

Khi viết test code cần phải hiểu rõ rằng mình muốn xác nhận, kiểm tra điều gì để từ đó có cách triển khai test code phù hợp. Ngoài ra tiêu để của test case cũng nên phản ánh rõ mục đích của nó:
- `given`: điều kiện tiên quyết (optional)
- `when`: hành động, thao tác
- `then`: giá trị kì vọng
là 3 yếu tố nên có trong tiêu đề của test case.

### [Test] Mutation Method

Với mutation method (method thay đổi trạng thái của entity), ta thử test postpone method như sau:

```TS
class Task {
  // ...
  
  postpone() {
    if (this.postponeCount > MAX_POSTPONE_COUNT) {
      throw DomainException("Max postpone count is overflow");
    }
    this.dueDate = dueDate.plusDays(1);
    this.postponeCount += 1;
  }
}
```

Với những methods có phát sinh ngoại lệ như thế này, ta cần test 2 TH:
- Normal case: khi không phát sinh ngoại lệ
- Unusual case: phát sinh ngoại lệ

Với case phát sinh ngoại lệ ta có thể sử dụng `reconstruct` method để đưa giá trị cần kiểm tra tới một giới hạn nhất định. Tuy nhiên cách làm này có nhược điểm là `reconstruct` method thường sẽ không có validation nên nếu sử dụng sai mục đích sẽ dẫn tới những lỗi tiềm tàng sau này.

### [Test] Reconstruct method

Reconstruct method là method dùng để tạo entity dựa theo dữ liệu được lấy ra từ DB.

```TS
class Task {
  // ...
  
  reconstruct(id: TaskId, name: TaskName, userId: UserId, status: TaskStatus, dueDate: Date, postponeCount: number) {
    return new Task(
      id, name, userId, status, postpone, dueDate,
    );
  }
}
```

`reconstruct` method cũng gọi đến `private method`, vậy nên từ bên ngoài class nếu muốn tạo instance thì chỉ có cách gọi đến `create` hoặc `reconstruct` method.

Bản thân `reconstruct` method chỉ có nhiệm vụ lắp các giá trị từ tham số vào thuộc tính tương ứng nên xử lí của method này là khá đơn giản. Việc test nó có thể cân nhắc giản lược.

### Value-Object Test

Xét VO TaskName như sau:

```TS
class TaskName {
   static MAX_LENGTH = 10;

  constructor(value: string) {
    if (value.length > MAX_LENGTH) throw DomainException("Task name length must be less than 10");
  }
}
```

Khi test class này ta cũng cần đưa ra 2 giá trị:
- value.length = 10: giới hạn trên của logic check (TH này không có lỗi)
- value.length = 11: TH này có lỗi

### Tách các Value-Object thành các class độc lập để test dễ dàng hơn

Khi ngoài task name, thì một Task object còn nhiều thuộc tính khác như: `dueDate`, `postponeCount`, ... Nếu không tách class riêng mà viết toàn bộ test vào trong Task class sẽ dẫn đến việc Test Class sẽ phình to làm giảm đi tính bảo trì của code.

Đồng thời việc tách riêng các VO cũng sẽ giúp cho nhiệm vụ của từng thuộc tính trở nên rõ ràng hơn, tránh tình trạng phải test những thứ không liên quan đến thuộc tính khi viết toàn bộ test vào một class duy nhất.

### Use-Case Test

Với use-case test ta sẽ đi từ `unit test` cho đến `combined test`.

Với repository ta có thể tiến hành mock chúng để dữ liệu trả về dưới dạng on-memory để từ đó không cần phải kết nối tới DB làm gì cả.

Đồng thời do ta đã test các method của entity rồi nên ở đây ta không cần thiết phải `assert` các thuộc tính của entity nữa.

### Object chuyên dụng để tạo instance cho mục đích test

Khi số lượng test cases tăng lên đồng nghĩa với việc số lượng các `entity` và `VO` cũng sẽ tăng lên theo. Từ đó dẫn đến việc cần thiết phải có một object chuyên dụng dùng để tạo ra các `entity` và `VO`

```TS
class TestTaskFactory {
  static create(id: TaskId, name: TaskName, userId: UserId, status: TaskStatus, dueDate: Date, postponeCount: number) {
    return new Task.reconstruct(
      id, name, userId, status, postpone, dueDate,
    );
  }
}
```

Để tăng sự tiện lợi ta cũng có thể sử dụng `default parameter value`

### Ưu điểm của unit test

Có 2 ưu điểm chính như sau:
- Tăng tốc độ thực thi test: vì toàn bộ dữ liệu test được thực thi on-memory, không cần phải kết nối tới DB từ đó giúp đẩy nhanh tốc độ test đặc biệt là khi số lượng test cases dần tăng lên theo thời gian.
- Giảm chi phí chuẩn bị dữ liệu: tuỳ vào nội dung test case mà ta cần chuẩn bị dữ liệu test cho phù hợp, chưa kể đến trường hợp có sử dụng foreign key thì việc chuẩn bị dữ liệu test sẽ rất mất thời gian. Với Mocking ta chỉ cần một vài dòng là có thể hoàn tất được khâu chuẩn bị dữ liệu test, từ đó giảm đi chi phí thực thi. Khi chi phí thực thi tăng lên, chúng ta sẽ "ngại" không muốn test, từ đó sẽ dẫn đến việc ảnh hưởng tới chất lượng chung của sản phẩm.

### Test Repository

Với Repository thì việc thực thi unit-test là không cần thiết vì ta cần phải xác minh dữ liệu nên với repository ta cần phải thực hiện `Combined test`.

### Test insert method & search method 

Với các `insert` method hoặc `save` method ta cần đưa chúng vào trong `Transaction` để thực hiện. Khi `assert` kết quả test ta cần check sự tồn tại của bản ghi mới được đưa vào trong DB.

```TS
await Transaction.manage(transaction => taskRepository.save(taskEntity, transaction));
```

### Object chuyên dụng để tạo test data

Khi tiến hành thực thi `CombinedTest` ta cần dữ liệu dùng cho nhiều test case khác nhau, khi đó việc sở hữu một object chuyên dụng để tạo test data sẽ rất thuận tiện. Ta có thể lấy ví dụ như sau:

```TS
class UserTestDataCreator {
  constructor(private readonly userRepository: UserRepository) {}

  create(userName = "user1") {
     const user = TestUserFactory.create(userName);
     userRepository.insert(user);
     return user; 
  }
}
```

Ở đây ta thấy method create tạo entity rồi truyền entity vào trong method insert của repository. Việc làm này có 2 lợi điểm:
- Che dấu đi được cấu trúc của table
- Hạn chế việc phát sinh các dữ liệu bất đồng bộ

Việc sử dụng SQL hoặc CSV để chuẩn bị dữ liệu test sẽ có những nhược điểm sau:
- Khi số test case tăng thì lượng dữ liệu cũng sẽ tăng dẫn tới chi phí bảo trì cho dữ liệu test cũng sẽ tăng
- Ngoài ra khi câu trúc table thay đổi, ta bắt buộc phải sửa SQL hoặc CSV file

Vậy nên việc sử dụng repository này sẽ rất có ích khi table thay đổi thì mọi sự ảnh hưởng từ sự thay đổi đó đều được repository "hấp thụ" và "xử lí" cho chúng ta từ đó sẽ giúp cho dữ liệu test không hề bị ảnh hưởng.

Việc tái sử dụng được code khi tiến hành test cũng sẽ hạn chế xung đột, bất đồng bộ dữ liệu ở một mức độ nào đó. Nếu ta sử dụng `reconstruct` method để tạo entity thì có thể dẫn đến trường hợp:
- Thêm nhầm code cho entity
- Quên thêm dữ liệu cho bảng con
Vậy nên việc tái sử dụng lại các method create của factory sẽ hạn chế việc phát sinh xung đột dữ liệu.

Việc chuẩn bị `Test data create object` ban đầu sẽ tốn nhiều thời gian xong vì ta có thể tái sử dụng nó ở nhiều test case khác nên về lâu dài nó sẽ giúp giảm đi chi phí thực thi test.
 
### Những vấn đề thường gặp khi thực hiện test - Sử dụng mock

Nhiệm vụ của `Usecase class` đó là sử dụng các public method của tầng domain để thực thi một nghiệp vụ nào đó. Với các method mà từ usecase class ta gọi ra ta sẽ tiến hành mock nó. Như ví dụ sau

```TS
class UpdateTaskStatusUseCase {
  constructor(private readonly taskRepository: TaskRepository) {}

  execute(taskId: TaskId) {
    const task = this.taskRepository.findById(taskId);

    if (!task) throw new Exception("Can not find task");
    task.postpone();

    await Transaction.manager(transaction => this.taskRepository.update(task, transaction));
  }
}
```

Ở đây ta chỉ cần quan tâm đến `What` đó là cần sử dụng method nào của Repository. Còn `How` sẽ là việc triển khai method đó ở dưới tầng Infra - đây là việc mà ta không cần phải quan tâm. Nên ở đây ta sẽ mock `TaskRepository` và 2 method `findById` và `update` của nó bằng cách cho trả về một giá trị nào đó.

Với ví dụ dưới đây

```TS
class BadUpdateTaskStatusUseCase {
  constructor(private readonly taskRepository: TaskRepository) {}

  execute(taskId: TaskId) {
    this.taskRepository.postponeTask(taskId);
  }
}
```

Đây là một ví dụ tồi là vì `postponeTask` ngoài việc thực hiện `find task` và `update task` còn phải thực thi luôn cả `domain method` là `postpone task`. Nên khi mock ta phải mock thêm cả domain method. Lúc này việc thực thi usecase test sẽ không còn nhiều ý nghĩa nữa do repository đang đảm nhận quá nhiều nhiệm vụ cùng một lúc -> Đây là một lỗi trong khi thiết kế hệ thống.

> Khi tiến hành test bằng mock ta cũng có thể phát hiện ra những lỗi trong thiết kế để có thể kịp thời refactor lại thiết kế và mã nguồn

### Liệu có cần viết test đối với việc gọi các external API 

 Với test cho external API, ta vẫn viết ở đơn vị `Unit Test` như thường, xong ta không nhất thiết phải `assert` các giá trị khi tiến hành viết test, việc gọi đến external API chỉ đơn giản coi như là `Chạy một script khởi động` mà thôi.

```TS
interface TwitterClient {
  postTweet(tweet: Tweet);
}

class TwitterSdkClient implements TwitterClient {
  function postTweet(tweet: Tweet) {
    // Use library to post tweet to twitter
  }
}

class TwitterSdkClientTest {
  function postTweet() {
    const client = TwitterSdkClient();
    client.postTweet(new Tweet("Hello World"));

    // Don't need to assertion
  }
}
```

Việc viết test code như vậy là do khi ta viết client class nhỏ nhất có thể thì sẽ giúp tăng tốc độ dev dự án. Ngoài ra `Unit test` sẽ gọi trực tiếp đến client class chứ không cần thiết phải khởi động app làm gì cả.

Ngoài ra ta cũng không cần thiết phải test external API trên tổng thể làm gì cả vì thông thường nếu setup trên CI cần phải test external API calling thì trong trường hợp external API không hoạt động hoặc việc gửi req có vấn đề dẫn đến phải gửi đi gửi lại nhiều req sẽ khiến cho CI failed. Do đó ta vẫn tạo test nhưng không thực thi nó trên tổng thể.

