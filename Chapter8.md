## Chương 8: Architecture

### Xử lí lỗi - Có nên trả về trực tiếp error message để cho client hiển thị ?

Trong trường hợp phát sinh lỗi, ta có 2 cách xử lí như sau:
- Trả về mã lỗi (error code) cho client, client sẽ dựa theo mã lồi mà hiển thị error message sao cho phù hợp
- Trả về error message trực tiếp cho client.

Việc trả về error message trực tiếp sẽ dễ dàng khi triển khai, tiếp kiệm thời gian, chi phí triển khai xong nếu có thay đổi về message thì ta phải thay đổi trực tiếp server code.

Còn nếu muốn 2 team server, frontend làm việc độc lập với nhau thì hãy trả về error code, khi đó nếu cần thay đổi message có thể ta cần thay đổi ở cả phía client lẫn server.

### Với các exception thì như thế nào

Ta sẽ định nghĩa Exception Handler dùng chung ở tầng presentation, tại đó ta sẽ bắt các exceptions rồi trả về error response object cho phía client.

Lấy ví dụ về `DomainException` như sau:

```TS
class DomainException {
  errorMessage: string;
  errorCode: ErrorCode;

  constructor(errorMessage: string, errorCode: ErrorCode) {
    this.errorMessage = errorMessage;
    this.errorCode = errorCode;
  }
}
```

2 thông tin tối thiểu mà exception cần lưu giữ đó là `error message` và `error code`.

Trong trường hợp có nhiều loại ngoại lệ ta có thể làm như sau:

```TS
class ExceptionHandler {
  function handleUsecaseException(e: UsecaseException) {} // catch Usecase Exception only
  function handleOtherException(e: Exception) {} // catch parent Exception class
}
```

### Client class access đến external API thì nên viết ở đâu

Với Interface của class access đến external API thì ta sẽ viết ở tầng Domain, còn phần triển khai sẽ được viết ở tầng Infrastructure. Request và Response type sẽ được định nghĩa ở tầng domain. Lấy ví dụ với việc gửi thông báo tới Slack.

```TS
// Domain layer
class Notification {
  targetChannelId: ChannelId;
  message: string;
}

// Abstract notification, so we can use Slack or Chatwork, etc ...
interface NotificationClient {
  notify(notification: Notification);
}
```

```TS
// Infrastructure layer
// In this layer we can implement for slack.

class SlackNotificationClient implements NotificationClient {
  function notify(notification: Notification) {
    // Send notification to slack logic
  }
}
```

### Sử dụng các thông tin liên quan tới Authentication như thế nào ?

Định nghĩa `interface` của `UserSession` tại tầng `use-case`

```TS
interface UserSession {
  userId: UserId;
  userRole: UserRole;
}
```

Nó sẽ được sử dụng tại một use-case nào đó như sau:

```TS
class SomeRequireSessionUseCase {
  execute(userSession: UserSession) {
    // using userSession logic
  }
}
```

Do ta truyền `userSession` vào use-case dưới dạng param, điều này sẽ làm cho việc thực hiện unit test bằng mock dễ dàng hơn.

```TS
class MockUserSession {
  userId: UserId;
  userRole: UserRole;

  constructor(userId = new UserId("1"), userRole = UserRole.NORMAL) {
    this.userId = userId;
    this.userRole = userRole;
  }
}

class SomeRequireSessionUseCaseTest {
  function test() {
    const userSession = new MockUserSession(null, UserRole.ADMIN);

    const usecase = new SomeRequireSessionUseCase();
    usecase.execute(userSession);
  }
}
```

Như vậy nếu ở tầng use-case ta cần phải sử dụng đến user session thì ta cũng không cần phải quan tâm đến thư viện dùng cho authentication là thư viện gì cả. Mọi thứ liên quan đến thư viện này đều được uỷ thác cho tầng `presentation`.

```TS
interface UserSessionProvider {
  getUserSession(): UserSession;
}

class SomeRequireSessionController {
  constructor(
    private readonly someRequireSessionUseCase: SomeRequireSessionUseCase,
    private readonly userSessionProvider: UserSessionProvider,
  ) {}  

  doSomething() {
    const userSession = userSessionProvider.getUserSession();
    someRequireSessionUseCase.execute(userSession);
  }
}
```

Ta không sử dụng `UserSessionProvider` trực tiếp ở trong use-case mà ta truyền `userSession` lấy về được vào trong use-case từ đó giúp việc viết test trở nên dễ dàng hơn.

Dưới đây sẽ là class thực thi `UserSessionProvider` được viết ở tầng `presentation`.

```TS
class UserSessionProvider implements UserSessionProvider {
  getUserSession() {
    // get session logic
  }
}
```

Việc chỉ sử dụng `UserSessionProvider interface` trong controller cũng nhằm mục đích viết test cho controller được dễ dàng.

### Liệu có thể sử dụng User class để lưu các thông tin liên quan đến Authentication

Nên tách thành 2 class riêng, lí do là vì khi kết hợp user profile information và authenticate information vào một class vô tình có thể làm cho class đó bị phình to. Khi đó ta cần phải chia user entity cũng như tách Aggregate. Điều này dẫn đến việc refactor sẽ khá tốn thời gian. Cho nên ngay từ quá trình modeling, ta nên cân nhắc có chia nhỏ class hay không.

<img width="665" alt="Screen Shot 2022-04-02 at 18 41 36" src="https://user-images.githubusercontent.com/15076665/161377421-836c3aac-7ae6-4ef8-9ce6-91a3e34f4ec7.png">

### Query Model (Lightweight CQRS) ta nên trả về user list như thế nào ?

CQRS ngoài việc thiết lập các model dùng cho việc query còn có:
- Phân chia data source
- Lưu trữ data
Tất cả đều được thực hiện bằng event driven.

Với chỉ riêng Query model ta có thể gọi đó là `Lightweight CQRS`.

Với các model dùng cho mục đích lấy về list data ta không sử dụng thuật ngữ `Aggregate` mà sử dụng thuật ngữ `Query Model` nhằm phân biệt với `Domain Model` - model dùng cho việc update, insert data.

Repository chỉ lấy vào cũng như đưa ra một Aggregate duy nhất nên việc đưa vào hoặc xuất ra nhiều Aggregate cùng một lúc là điều không được phép.

Object dùng để lấy dữ liệu thông qua `Query Model` gọi là `QueryService`.

Cùng xét 1 ví dụ cụ thể như sau:

<img width="428" alt="Screen Shot 2022-04-02 at 21 56 11" src="https://user-images.githubusercontent.com/15076665/161384342-972dabaf-9591-44e5-8c4d-7a92a4888664.png">

Trong trường hợp ta cần lấy về `userName` cùng với `departmentName` của user đó, ta sẽ định nghĩa `QueryModel` - `Dto` như sau:

```TS
class UserAndDepartmentDto {
  userId: UserId;
  userName: string;
  departmentName: string;
}

interface UserAndDepartmentDtoQueryService {
  fetchList(): UserAndDepartmentDto[];
}
```

`QueryModel` - `Dto` sẽ khác nhau tuỳ theo use-case nên nó sẽ được định nghĩa ở tầng use-case. Còn class implement `DtoQueryService` sẽ nằm ở tầng infra.

```TS
class UserAndDepartmentDtoQueryService {
  fetchList() {
    // get user & department logic
  }
}
```

`QueryService` sẽ được gọi trong `use-case class` hoặc trong controller. Tuy nhiên nếu giá trị trả về từ `QueryService` cần phải chỉnh sửa hãy gọi từ `use-case`.

Sử dụng `QueryModel` không phải là cách duy nhất khi ta muốn lấy thông tin từ nhiều `Aggregate`, một cách khác ta cũng có thể cân nhắc đó là sử dụng các `Repository methods` bên trong use-case class, sau đó tiến hành `reconstruct` lại kết quả trả về.

Sử dụng `QueryModel` sẽ có những ưu, nhược điểm như sau:
1. Ưu điểm:
- Đơn giản hoá việc lấy dữ liệu từ nhiều `Aggregate` cùng một lúc, qua đó tăng tính bảo trì cho mã nguồn
- Dễ dàng cải thiện performance hoặc tiến hành query tuning.
- Có thể dễ dàng tiến hành paging hoặc filter
2. Nhược điểm:
- Phức tạp hoá architecture, mất thời gian khi tìm hiểu
- Khó truy vết những chỗ đang tham chiếu tới domain object data

### Nếu sử dụng QueryModel thì mọi xử lí liên quan đến query data đều bắt buộc phải đi qua QueryModel ?

Nếu tuân thủ nghiêm ngặt CQRS thì việc đó là cần thiết. Xong nếu việc này làm tăng chi phí dev, ta cũng có thể cân nhắc sử dụng `Repository` thay vì lúc nào cũng thông qua `QueryModel`.

> Không nên quá cứng nhắc trong việc phân chia `Update Relate` hoặc `Query Relate` mà hãy lựa chọn giải pháp phù hợp nhất cho hệ thống của bạn

### QueryService nên chứa bao nhiêu domain knowledge là đủ ?

Có những trường hợp khi ta chỉ định điều kiện cho QueryService thì vô tình ta có thể làm lộ lọt knowledge của tầng domain / use-case xuống tầng infra. Xét ví dụ sau:

```TS
interface ITaskQueryService {
  findRemindTarget(userId: UserId): TaskDto[];
}

class FetchRemindTargetUseCase {
  execute(userSession: UserSession) {
    return taskQueryService.findRemindTarget(userSession.userId);
  }
}

class TaskQueryService implements ITaskQueryService {
  findRemindTarget(userId: UserId) {
    // SQL condition: status = NOT_FINISHED, ...
  }
}
```

Rõ ràng ở đây ta thấy nghiệp vụ lấy các task vẫn chưa hoàn thiện để remind cho người dùng là của phía use-case nhưng nó đã bị lộ lọt xuống tầng infra.

Để tránh tình trạng này xảy ra, ta sẽ chỉ ra các điều kiện để fetch dữ liêu thông qua các params như sau:

```TS
interface ITaskQueryService {
  findByStatusAndDueDate(userId: UserId, status: TaskStatus, dueDate: Date): TaskDto[];
}

class FetchRemindTargetUseCase {
  execute(userSession: UserSession) {
    return taskQueryService.findByStatusAndDueDate(
      userSession.userId, TaskStatus.NOT_FINISHED, new Date()
    );
  }
}
```

### DTO class thể hiện giá trị trả về của use-case nên được định nghĩa ở đâu ?

Nên định nghĩa ở cùng một folder / package hoặc cùng một file với use-case class. Vì DTO class thể hiện giá trị trả về của use-case , vậy nên hãy định nghĩa nó ở gần use-case class.

```shell
task/
  FetchTaskUseCase.ts
  FetchTaskUseCaseDto.ts
```

Không nên tách ra thành một folder DTO riêng như dưới đây:

```shell
usecase/
  FetchTaskUseCase.ts
  FetchUserUseCase.ts
dto/
  FetchTaskDto.ts
  FetchUserDto.ts
```

Việc xếp folder như vậy sẽ làm cho việc truy vết mối liên hệ giữa use-case và dto trở nên khó khăn trong khi không đem lại lợi ích to lớn nào cả.

### Có nên viết hàm chuyển format của DTO cho mục đích hiển thị hay không ?

Câu trả lời là không. Việc hiển thị là của tầng presentation, không hề liên quan đến tầng use-case. Nếu muốn viết hàm chuyển format cho mục đích hiển thị thì nên viết ở phía tầng presentation. Tầng này sẽ nhận DTO từ use-case rồi chuyển format dựa theo yêu cầu hiển thị.

Mọi thứ ở tầng presentation không nên làm ảnh hưởng đến tầng use-case

> Hãy nhận thức rõ nhiệm vụ của từng class để tránh viết những xử lí ảnh hưởng đến nhiệm vụ của class đó

### Liệu có nên sử dụng class của tầng domain cho giá trị trả về của use-case hay không ?

Có thể sử dụng nhưng nó vi phạm quy tắc về nghiệp vụ của từng tầng do đó điều không nên áp dụng cách triển khai này vào thực tế. 

Lấy ví dụ như sau:

```TS
class TaskController {
  async fetchTask(usecase: FetchTaskUseCase) {
    const task = await usecase.execute();

    return new TaskView (
      id: task.id,
      dueDate: task.displayDate(),
    );
  }
}

class TaskEntity {
  dueDate: Date;

  displayDate(): string {
    return dueDate.format('yyyy/mm/dd');
  }
}
```

Rõ ràng ta thấy tại class entity, đang có một hàm chuyển định dạng của `dueDate` nhằm mục đích phục vụ nhiệm vụ trả về cho client ở tầng `presentation`. Điều này không hẳn sai, nhưng khi client yêu cầu nhiều loại định dạng hơn thì số lượng method chuyển định dạng ở entity class (tầng domain) sẽ tăng lên.

Hơn nữa nhiệm vụ chuyển định dạng này là nằm ở tầng `presentation` chứ không phải ở tầng `domain`.

Nếu quyết định viết code theo hướng này thì ngay từ ban đầu cần có sự đồng thuận trong team về tiêu chuẩn viết code.

### Có nên tạo instance của entity hoặc value-object ở tầng presentation rồi truyền vào tầng use-case hay không ?

Về nguyên tắc thì việc tạo instance của entity hay value-object chỉ nên được tiến hành ở tầng use-case.

Vì nhiệm vụ của tầng use-case chính là:

> Từ giá trị đầu vào, tạo ra domain object, sử dụng repository để lưu vào trong DB

Nếu tiến hành 1 phần công việc của usecase trong presentation thì sẽ:
- Giảm đi sự kết nối giữa 2 tầng này
- Tăng tính phụ thuộc giữa chúng

Tuy nhiên ta cũng có thể chấp nhận trường hợp tạo value-object tại tầng presentation.

### cron job nên được đặt ở tầng nào ?

Nên đặt ở tầng presentation vì ta không muốn usecase phải quan tâm đến scheduler làm những nhiệm vụ gì hay cron job làm những nhiệm vụ gì.

Các cron job này sẽ định kì gọi đến phần xử lí của usecase vì thế nên đặt các cron job này ở phía controller nhận các HTTP req.

### Trong onion architect, việc phụ thuộc vào library hay framework chỉ nên được cho phép đến tầng nào ?

Trên thực tế ta không hề muốn tầng:
- Domain
- Usecase

bị phụ thuộc vào framework hay library, ở các tầng này ta chỉ nên sử dụng interface khi gọi đến các repository, còn class implement interface này sẽ được đặt ở tầng infrastructure.

Thế nhưng phổ biến hiện nay ở các web app đó là DI (dependency injection), transaction. Với các framework hiện đại thì việc sử dụng DI hay transaction không còn là điều gì đó quá khó khăn nữa cả.

VD với Spring framework của Java, ta chỉ cần thêm annotation cho class là xong. Việc này rất đơn giản nên ta có thể áp dụng nó cho tầng domain, usecase đều được do mức độ ảnh hưởng của tech lên hai tầng này là không nhiều.

Ngược lại việc triển khai:
- Endpoint cho API (controller)
- Request path

lại phụ thuộc rất nhiều vào framework đi kèm. Do đó những việc triển khai này chỉ nên dừng ở mức độ của tầng presentation, đừng để nó lan rộng đến tầng usecase.

### Trong onion architecture, nên quản lí các xử lí transaction như thế nào ?

Một transaction sẽ bắt đầu khi vào usecase, nếu kết thúc bình thường thì commit, còn nếu xuất hiện ngoại lệ thì rollback.

Việc quản lí các xử lí liên quan đến transaction tại usecase sẽ phụ thuộc vào framework.

### Có nên gọi usecase từ một usecase ?

Không nên. Vì nếu cho phép điều này sẽ dẫn tới một chuỗi các lời gọi giữa các usecases (`usecase1` → `usecase2` → `usecase3` ...). Điều này sẽ làm cho việc truy vết code sau này sẽ rất khó khăn.

Vì thế nên:
- Từ controller chỉ gọi duy nhất 1 use-case mà thôi.
- Với use-case được gọi từ nhiều use-cases khác thì ở tầng use-case ta nên tách thành 1 class độc lập để các use-case cùng tham chiếu tới.

Như hình dưới đây, với các xử lí hay được gọi như fetch data hay convert kiểu dữ liệu thì ta nên tạo các classes dùng chung như `XxxFetcher` hay `YyyConverter`.

<img width="560" alt="Screen Shot 2022-06-04 at 12 46 08" src="https://user-images.githubusercontent.com/15076665/171980851-11aac2d9-8d32-4ba6-8882-182fd8640f95.png">

Thiết kế như phía trên ngoài khả năng tái sử dụng cao, nó còn:
- Tăng tính kết nối giữa class độc lập với các use-cases
- Tăng tính dễ đọc
- Tăng tính dễ test

### Tại tầng domain, object được tham chiếu bởi nhiều Aggregate thì nên đặt ở package nào ?

Có 2 cách:
① Đặt ở package mà object đó được định nghĩa
② Đặt ở một package dùng chung

Lấy ví dụ cho ① `UserStatus` là một object, được sử dụng dưới dạng property của `User Entity`, trong `Task Entity` có một hàm xử lí sử dụng đến `UserStatus`. Ta thấy rằng trong User Aggregate thì UserStatus được định nghĩa và UserStatus được tham chiếu bởi Task Aggregate. Với TH này ta sẽ đặt `User` và `UserStatus` trong cùng `folder user`.

Với ②, ta thấy rằng đây là những object có tính trừu tượng và không được định nghĩa trong một Aggregate cụ thể nào cả. Cho TH này ta sẽ định nghĩa trong một `package shared`.

Với các objects được tham chiếu bởi nhiều Aggregate, nếu ta chỉ theo 1 công thức là đặt chúng vào một package chung thì ranh giới giữa `bên định nghĩa` và `bên tham chiếu` sẽ không thể thấy rõ được.

> Vậy nên hãy nhận thức rõ ràng sẽ định nghĩa object ở đâu để biết cách đặt package sao cho phù hợp

### Business logic là gì ?

Business logic là một khái niệm rộng, cho nên ta sẽ tiến hành cắt nghĩa thành 2 khái niệm con như sau:
- Domain knowledge: là các quy tắc, luật của một lĩnh vực mà ta sử dụng software để giải quyết vấn đề. Kể cả khi không có software thì nó vẫn tồn tại, kể cả khi use-case thay đổi thì nó cũng không thay đổi. Được biểu diễn bằng sơ đồ Domain Model
- Usecase: là các hành động được tạo bởi người dùng thông qua software. Nếu không có software thì nó không hề tồn tại. Được biểu diễn bằng sơ đồ use-case
- Display knowledge: data-format, quy tắc lúc hiển thị, ...

Theo cách phân chia như trên, sẽ có một sự tương ứng vai trò của các tầng từ:
Domain → Usecase → Presentation

> Dựa theo nội dung của xử lí, hãy viết xử lí vào tầng thích hợp

### Liệu với một dự án, có thể kết hợp giữa DDD pattern và Transaction script được hay không ? 

Với các dự án phát triển từ zero ta chỉ nên áp dụng **MỘT** pattern duy nhất mà thôi.

Với những dự án sẵn có, khi áp dụng DDD cho một phần của dự án, ta cần có:
- Quy chuẩn, quy tắc khi thực thi
- Tài liệu hoá các quy tắc trên

> Nếu không có nhưng quy tắc, luật chung thì cách viết code của mỗi người mỗi khác sẽ làm cho hệ thống trở nên rắc rối

### Entity của DDD có gì khác so với entity của Clean Architecture ?

Trong DDD thì entity được định nghĩa ở tầng domain. Còn trong clean architecture thì `tầng entity` sẽ định nghĩa các business rule dùng cho toàn hệ thống (Enterprise wide).

Trong DDD, nếu trong một hệ thống to có sự khác nhau về context thì model sẽ được phân chia rõ ràng cho từng context khác nhau. Điều này khác với quy tắc "business dùng cho toàn bộ hệ thống".

Do về mặt tư tưởng có ít nhiều sự khác biệt nên khi áp dụng Clean Architecture cho DDD thì `Tầng Entity` có thể chuyển thành `Tầng Domain`.

### Liệu có nên định nghĩa interface nếu chỉ có duy nhất một class implement nó hay không ? 

Nhờ có interface, ta có thể kiểm soát được mối quan hệ phụ thuộc giữa các layer khác nhau để từ đó vẫn đảm bảo được vai trò của từng tầng.

Lấy ví dụ với Onion Architecture, hãy xem xét tới mối liên hệ giữa:
- Repository Interface
- Repository Implement Class
- Use-case class

<img width="626" alt="Screen Shot 2022-06-04 at 16 57 09" src="https://user-images.githubusercontent.com/15076665/171990483-cdfffc99-5c1f-4b7b-93c2-d5910e22ee67.png">

Như hình trên ta thấy, tầng domain hay tầng use-case đều không có bất kì một ý niệm gì về infra class của repository, từ đó tránh được mối quan hệ phụ thuộc giữa chúng. Không những thế ta còn có thể nhận ra được những lợi ích như sau:
- Tính dễ đọc
- Tính dễ test

đều được tăng lên.

### Liệu có thể định nghĩa FIRST CLASS COLLECTION bao lấy một list các entites tại tầng domain ?

Hoàn toàn có thể với ví dụ về định nghĩa class UserList để hiển thị list các user.

Tuy nhiên nếu trong trường hợp collection này có thể được dùng cho:
- Hiển thị list user
- Một thao tác khác liên quan đến user list

thì không nên định nghĩa ở tầng domain vì bản chất nó thuộc về từng use-case khác nhau. Do đó nên định nghĩa nó ở tầng use-case.

Tầng domain không nên phụ thuộc vào use-case, nên nếu định nghĩa ở tầng domain thì nên định nghĩa với một cái tên trừu tượng "tập hợp các users".

Các methods bên trong class cũng sẽ mang tính trừu tượng cao do nó không phụ thuộc vào bất kì use-case nào.

### Có nên cho phép entity immutable hay không ?

Hoàn toàn có thể. Nếu như vậy entity class sẽ trông như sau:

```TS
class TaskEntity {
  done(): TaskEntity {
    const task = new TaskEntity(this.id, this.name, TaskStatus.DONE);
  }
}
```

method `done()` sẽ không thay đổi trực tiếp các thuộc tính của entity mà trả về một entity instance mới.
