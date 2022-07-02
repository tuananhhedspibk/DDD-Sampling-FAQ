## Chương 5: Kết tập

### Cách đảm bảo đồng bộ hoá giữa các Aggregate

Lấy một ví dụ về app quản lí task. Khi ta tiến hành tạo task thì `Aggregate Task` sẽ được tạo, đồng thời với đó là `Aggregate ActivityHistory` (lịch sử hoạt động) cũng sẽ được tạo theo. Nếu 2 aggregates này không đồng bộ hoá, thống nhất với nhau thì sẽ gây ra rất nhiều vấn đề.

Ngoài `Task Aggregate` ra ta còn có thể thấy rằng các `Aggregates` khác khi được tạo ra cũng sẽ kéo theo `ActivityHistory` được tạo. Như ví dụ dưới đây:

<img width="593" alt="Screen Shot 2022-03-13 at 21 17 45" src="https://user-images.githubusercontent.com/15076665/158059047-b61d67f9-dca1-4949-a0a2-109c023e2431.png">

Có thể có 3 cách triển khai như sau:
1. Cập nhật nhiều Aggregate tại use-case cùng 1 lúc
2. Sử dụng domain-service
3. Sử dụng domain-event

### Cập nhật nhiều aggregates cùng 1 lúc tại use-case

Với cách làm này ta sẽ tiến hành tạo nhiều aggregates tại use-case cùng một lúc, sau đó truyền vào repository một cách lần lượt

```TS
class Task {
  name: string;
}

abstract class ITaskRepository {
  insert(task: Task);
}

class ActivityHistory {
  constructor() {}
  static createFromTask(task: Task) {
    return ActivityHistory(`Create From ${task.name}`);
  }
}

abstract class IActivityHistoryRepository {
  insert(activityHistory: ActivityHistory);
}

// Usecase
class CreateTaskUseCase {
  execute(taskName: string) {
    const task = new Task(taskName);
    taskRepository.insert(task);

    const activityHistory = ActivityHistory.createFromTask(task);
    activityHistoryRepository.insert(activityHistory);
  }
}
```

Ưu điểm của cách làm này là khá đơn giản nên nó thường được áp dụng cho những người mới làm quen với DDD. Tuy nhiên nhược điểm của nó nằm ở chỗ nếu 1 use-case khác vô tình phá vỡ đi sự đồng bộ giữa `Task` và `ActivityHistory` sẽ khiến cho người đọc code không nhận ra rằng mối liên hệ giữa `Task` và ``ActivityHistory`

```TS
class WrongCreateTaskUseCase {
  execute(taskName: string) {
    const task = new Task(taskName);
    taskRepository.insert(task);
  }
}
```

### Sử dụng domain service

Việc gắn thêm chữ `Service` cho DomainService class name là một việc làm không được khuyến khích, nguyên nhân là vì nó khiến cho class đó không biểu thị rõ ràng nghiệp vụ mà nó đảm nhận.

```TS
class CreateTaskUseCase {
  constructor(taskCreator: TaskCreator) {}
}
```

Ở ví dụ trên thì `TaskCreator` là Domain Service. Việc đảm bảo tính đồng bộ giữa 2 Aggregates sẽ được chuyển qua `TaskCreator` chứ không phải thực hiện ở UseCase.

```TS
class TaskCreator {
  create(taskName: string) {
    const task = new Task(taskName);
    taskRepository.insert(task);

    const activityHistory = ActivityHistory.createFromTask(task);
    activityRepository.insert(task);
  }
}
```

Thế nhưng việc làm này cũng không thể phòng tránh được việc các use-case khác phá vỡ đi tính đồng bộ. Để giải quyết vấn đề này ta cần xem xét đến visualization của các class, interface (public / private) - việc điều chỉnh này sẽ tuỳ vào từng ngôn ngữ mà có sự khác biệt nhất định. Phần triển khai dưới đây là dành cho typescript.

```TS
class TaskEntity {
  constructor(taskName: string) {
    this.taskName = taskName;
  }

  create(param: TaskCreateParameter): TaskEntity {
    return TaskEntity(param.taskName);
  }
}
```

create method của TaskEntity class sử dụng `TaskCreateParameter` cho tham số đầu vào. Định nghĩa `TaskCreateParameter`  sẽ nằm trong `DomainService`.

```TS
public interface TaskCreateParameter {
  taskName: string;
}

private class TaskCreateParameterImpl implements TaskCreateParameter {
  taskName: string;
}

class TaskCreator {
  constructor(
    taskRepository: taskRepository,
    activityHistoryRepository: ActivityHistoryRepository
  ) {}

  create(taskName: string) {
    const task = Task.create(TaskCreateParameterImpl(taskName));
    taskRepository.insert(task);

    const activityHistory = ActivityHistory(task);
    activityHistoryRepository.insert(task);
  }
}
```

class `TaskCreateParameterImpl` dùng để truyền vào hàm create của `TaskEntity` class, nhưng nó được định nghĩa là private class do đó ở use-case khác nếu sử dụng `TaskCreateParameterImpl` sẽ dẫn tới lỗi biên dịch.

```TS
class CreateWrongTaskUsecase {
  execute(taskName: string) {
    const task = Task.create(TaskCreateParameterImpl(taskName));
    taskRepository.insert(task);
  }
}
```

Lúc này việc tạo task chỉ có thể được thực hiện trong `TaskCreator` mà thôi, qua đó tránh được vấn đề bất đồng bộ giữa các Aggregates.

Tuy nhiên cách làm này có nhược điểm là sẽ làm cho DomainService ngày một phình to trong khi đó DomainEntity và Value-Object sẽ ngày càng thu nhỏ lại (không còn chứa nhiều domain logic nữa). Qua đó sẽ dẫn tới việc cách viết code của chúng ta sẽ quay trở về thời kì "tiền DDD".

Để tránh điều này xảy ra có 2 biện pháp sau đây:
- Đầu tiên là giảm thiểu tối đa việc sử dụng DomainService, chỉ dùng nó khi thực sự cần thiết. Kể cả khi sử dụng nó, ta cũng không nên dồn quá nhiều domain logic vào đây.
- Ngoài ra không nên đặt tên theo format `ABCXYZService`, hãy đặt một cái tên phản ánh đúng nghiệp vụ mà nó đảm nhận, qua đó giúp ta có thể tránh việc thêm quá nhiều các methods thừa thãi khác vào class.

### Sử dụng DomainEvent

Chúng ta sẽ thử thêm `DomainEvent` vào sơ đồ domain model.

![Screen Shot 2022-03-19 at 12 32 14](https://user-images.githubusercontent.com/15076665/159105098-f3deee57-7fbb-44df-9645-d2d4cc7de638.png)

`ActivityHistory` sẽ không phụ thuộc vào `Task` nữa mà sẽ phụ thuộc vào `TaskCreatedEvent`. Nếu đi theo hướng này, flow thực thi sẽ như sau:
1. Tạo `DomainEvent` tại một thời điểm nhất định trong method xử lí của entity, lưu event vào tập hợp các events của entity
2. Sau khi method insert/ update của Repository thành công, sẽ Publish event.
3. EventListener sẽ bắt các event này và tiến hành xử lí các Aggregates khác. (việc bắt event sẽ phụ thuộc vào từng framework mà có cách viết, triển khai khác nhau).

```TS
class CreateTaskUseCase {
  execute() {
    const task = TaskEntity.create(taskName); // tại đây sẽ tạo domain event và lưu vào tập các events của entity
    taskRepository.insert(task, domainEventPublisher); // sau khi insert thành công, sẽ tiến hành publish event nhờ vào domainEventPublisher
  }
}

abstract class DomainEventPublisher {
  publish(event: DomainEvent);
}

class ActivityHistoryEventListener {
  @OnEvent()
  createActivityHistory(event: TaskCreatedEvent) {
    const activityHistory = ActivityHistory.fromTaskCreatedEvent(event);
    activityHistoryRepository.insert(activityHistory);
  }
}
```

Sử dụng DomainEvent sẽ có 3 ưu điểm sau:
1. Đảm bảo được tính đồng bộ giữa các Aggregates.
2. Usecase, EventListner có thể được triển khai một cách đơn giản hơn
3. Mối liên hệ giữa các Aggregates cũng sẽ được thấy rõ ràng hơn khi thông qua event.

Về ưu điểm thứ nhất ta thấy rằng khi tạo `Task` xong, việc tạo `ActivityHistory` bắt buộc phải thông qua `ActivityHistoryEventListener` nên sẽ tránh tình trạng use-case phá vỡ đi sự đồng bộ giữa các Aggregates.

Về ưu điểm thứ 2 ta thấy việc không phải giới hạn truy cập (public / private) cũng khiến việc triển khai đơn giản đi rất nhiều.

Tuy nhiên nhược điểm của cách này là ta cần phải giải thích chi tiết về nó cho các thành viên khác trong team.

Quá trình publish event được triển khai ở tầng infra sẽ trông như sau

```TS
class TaskRepository {
  insert(task: TaskEntity, domainEventPublisher: DomainEventPublisher) {
    task.getDomainEvents().forEach(event => domainEventPublisher.publish(event));
    task.clearEvents(); // Xoá mọi events sau khi đã publish xong
  }
}
```

Ta thấy ở đây cần truyền `domainEventPublisher` param vào `insert` method. Đây là một ngoại lệ vì về lí thuyết ta chỉ nên truyền `entity` vào `repository` mà thôi. Tuy nhiên nếu không thể hiện rõ thông qua việc truyền param thì sẽ rất khó truy vết được `event`. Ngoài ra còn có một sự lựa chọn khác đó là DI.

Tuy nhiên có một vấn đề đó là khi xem code của use-case ta không thể biết được `DomainEvent` được tạo khi nào. Dưới đây là một giải pháp:

```TS
class CreateTaskUseCase {
  constructor(
    private taskRepository: TaskRepository,
    private domainEventPublisher: DomainEventPublisher,
    private domainEventSeedFactory: DomainEventSeedFactory,
  ) {}

  execute(taskName: string) {
    const seed = domainEventSeedFactory.createSeed();
    const task = Task.create(taskName, seed);
    taskRepository.insert(task, domainEventPublisher);
  }
}
```

Ta cần truyền `DomainEventSeed` vào `create` method của `TaskEntity`.  Sẽ có câu hỏi đặt ra là "Liệu việc truyền DomainEventSeed vào có ảnh hưởng đến các Aggregate khác hay không ?".  Câu trả lời là chỉ cần xem các `DomainEvent` được tạo ra trong method là có thể biết được.

```TS
class DomainEvent {
  seed: DomainEventSeed;
}

public interface DomainEventSeed {}

private class DomainEventSeedImpl {}

class DomainEventSeedFactory {
  createSeed(): DomainEventSeed {
    return DomainEventSeedImpl();
  }
}
```

Việc đặt `DomainEventSeedImpl` là private class sẽ giúp hạn chế việc tạo `DomainEventSeed` lung tung ở nhiều chỗ,
khi đó việc tạo `DomainEventSeed` bắt buộc phải thông qua `DomainEventSeedFactory`.

### Có nên update nhiều Aggregates trong cùng 1 transaction hay không ?


Cũng có nhiều luồng suy nghĩ về vấn đề này. Trên thực tế việc update nhiều Aggregates trong cùng 1 transaction sẽ khiến cho phạm vi của transaction sẽ lớn,
từ đó dẫn tới việc nếu có lỗi sẽ gây ra ảnh hưởng trong một phạm vi lớn.

Tuy nhiên nếu cần thiết phải rollback thì việc update nhiều Aggregates trong cùng 1 transaction sẽ phát huy hiệu quả trông thấy rõ.

Do đó ta có thể cân nhắc việc update 1 hay nhiều Aggregates trong cùng 1 transaction.

### Nên thiết kế phạm vi của Aggregate như thế nào ?

Ta có thể xem xét ví dụ về "Trường học" và "Khoa" như dưới đây. Có cả thảy 2 pattern cho chúng ta có thể xem xét.

![Screen Shot 2022-03-19 at 18 35 07](https://user-images.githubusercontent.com/15076665/159115876-dd55c7dd-20e8-4ac9-8928-a6e1c6f1a6fa.png)

Với Pattern A ta thấy Aggregate có phạm vi rất lớn, nó có những ưu điểm sau:
- Đảm bảo được tính đồng bộ
- Việc thực thi cũng như test sẽ trở nên đơn giản hơn

Tuy nhiên Aggregate với phạm vi lớn cũng sẽ có những nhược điểm sau:
- Lượng data cần xử lí cũng sẽ tăng theo
- Cơ chế visualization cũng sẽ phức tạp hơn
