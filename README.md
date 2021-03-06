# SOLID 設計原則

高內聚，低偶合



- 單一原則，簡稱 SRP
    - 原文定義：There should never be more than one reason for a class to change
- 簡單來說：
    - 每個 class 只負責一個職責內容
    - 如果該 class 擁有多個職責，就應該把每個職責獨立成一個 class，再以組合方式成為一個 class
- 好處：
    - class 複雜度降底，清楚明白職責內容
    - 可讀性、可維護性提高
    - 當需求變更，引起的風險降低

**Bad:**

```csharp
public class AuthenticationService
{
    public bool Verify(int accountId, string password)
    {
        //取得使用者密碼
        var fromDbPassword = GetAccountPassword(accountId);

        //取得加密後的密碼
        var hashPassword = GetHashPassword(password);

        //驗證
        if (hashPassword.equals(fromDbPassword))
        {
            return true;
        }
        retun false;
    }

    private string GetAccountPassword(int accountId)
    {
        //從Db取回使用者密碼
        return "sdsiebq";
    }

    private string GetHashPassword(string password)
    {
        //加密過的密碼
        return "sdsiebq";
    }
}

```

**Good:**

```csharp
public class AuthenticationService
{
    private IProfileService _profileService;
    private IHashService _hashSerivce;

    public AuthenticationService(IProfileService profileService, IHashService hashService)
    {
        _profileService = profileService;
        _hashService = hashService;
    }

    public bool Verify(int accountId, string password)
    {
        //取得使用者密碼
        var fromDbPassword = this._profileService.GetAccountPassword(accountId);

        //取得加密後的密碼
        var hashPassword = this._hashService.GetHashPassword(password);

        //驗證
        if (hashPassword.equals(fromDbPassword))
        {
            return true;
        }
        retun false;
    }
}

```

- 開放封閉原則，簡稱 OCP
    - 原文定義：software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.
- 簡單來說：
    - 當一個 classes, modules, functions 碰到需求變更時，應該是增加 code，而不是修改原本的 code，來滿足需求
    - 要不改 code 是不太可能，而是要改組合或 mapping concreate class 的部分，而不是 concreate class 的內容
- 好處：
    - 可測試性
    - 重用性
    - 可維護性

**Bad:**

```csharp
public class AuthenticationService
{
    public bool Verify(int accountId, string password)
    {
        //取得使用者密碼
        var fromDbPassword = GetAccountPassword(accountId);

        //取得加密後的密碼
        var hashPassword = GetHashPassword(password);

        //驗證
        if (hashPassword.equals(fromDbPassword))
        {
            return true;
        }

        //寫 log
        LogInfo("驗證失敗");

        retun false;
    }

    private string GetAccountPassword(int accountId)
    {
        //從Db取回使用者密碼
        return "sdsiebq";
    }

    private string GetHashPassword(string password)
    {
        //加密過的密碼
        return "sdsiebq";
    }

    private void LogInfo(string message)
    {
        //寫 log
    }
}

```

**Good:**

```csharp

static void Main(string[] args)
{
    //原本是用 EventLogger
    //new AuthenticationService(new ProfileService(), new Sha256Service(), new EventLogger());

    //改成 NLogger 
    new AuthenticationService(new ProfileService(), new Sha256Service(), new NLogger());
}

public class AuthenticationService
{
    private IProfileService _profileService;
    private IHashService _hashSerivce;
    private ILogger _logger;

    public AuthenticationService(IProfileService profileService, IHashService hashService, ILogger logger)
    {
        _profileService = profileService;
        _hashService = hashService;
        _logger = logger;
    }

    public bool Verify(int accountId, string password)
    {
        //取得使用者密碼
        var fromDbPassword = this._profileService.GetAccountPassword(accountId);

        //取得加密後的密碼
        var hashPassword = this._hashService.GetHashPassword(password);

        //驗證
        if (hashPassword.equals(fromDbPassword))
        {
            return true;
        }

        //寫 log
        _logger.Info("驗證失敗");

        retun false;
    }
}

```

- 里氏替換原則，簡稱 LSP
    - 原文定義：
        - If for each object o1 of type S there is an object o2 of type T such that for all programs P defined in terms of T, the behavior of P is unchanged when o1 is substituted for o2 then S is a subtype of T.
        - Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it
- 簡單來說：
    - 子類別必須能夠替換父類別，並且功能不受影響    
- 目的是什麼：
    - 規範繼承的標準，因為繼承的特性導致高耦合，子類別對於方法修改(Override, Overload) 必須依照父類別行為方向，否則會對整體的繼承體系照成破壞，會有產生不可預測的行為與不好察覺 Bug
- 原則：
    - 子類別必須完全實現父類別的方法
    - 子類別可以擁有自己不同的屬性與方法
- 說明：
    - 以這例子，正方型無法實作父類的方法，而且要計算正確的面積，就需要改動父類別程式.

    
**Bad:**

```csharp

public static void Main(string[] args)
{
    //長方型面積
    var rectangle = new Rectangle(2, 5);
    Console.WriteLine("Rectangle Area : " + rectangle.GetArea());

    //正方型面積
    var square = new Square(2, 5);
    Console.WriteLine("square Area : " + square.GetArea());
}

internal class Rectangle
{
    private readonly int _width;
    private readonly int _height;

    public Rectangle(int width, int height)
    {
        _width = width;
        _height = height;
    }

    public int GetArea()
    {
        return _width * _height;
    }
}

internal class Square : Rectangle
{
    public Square(int width, int height) : base(width, height)
    {     
    }
}
```

**Good:**

```csharp

public static void Main(string[] args)
{
    //長方型面積
    var rectangle = new Rectangle(2, 5);
    Console.WriteLine("Rectangle Area : " + rectangle.GetArea());

    //正方型面積
    var square = new Square(2, 5);
    Console.WriteLine("square Area : " + square.GetArea());
}

internal abstract class ShapeBase
{
    protected int Width;
    protected int Height;
    public abstract int GetArea();
}

internal class Square : ShapeBase
{
    public Square(int width, int height)
    {
        base.Width = width;
        base.Height = height;
    }

    public override int GetArea()
    {
        if (base.Width == base.Height)
        {
            return base.Width * base.Height;
        }

        throw new Exception("長寛長度不相等");
    }
}

internal class Rectangle : ShapeBase
{
    public Rectangle(int width, int height)
    {
        base.Width = width;
        base.Height = height;
    }

    public override int GetArea()
    {
        return base.Width * base.Height;
    }
}

```

- 最少知識原則，簡稱 LoD/LKP
    - 原文定義：
        - Law of Demeter, LoD (狄米特法則)
        - Least Knowledge Principle, LKP (最少知識原則)
- 簡單來說：
    - 任何一個 object 應該只要讓外部 object 知道，最少且缺一不可的資訊，就可以正常 interact
- 目的是什麼：
    - 用來解耦合，也就是降低類別與類別之間耦合的程度。當耦合程度降低時，每一個class可以跟其他class互動的機會就會增加，reuse的機會就會增加
- 原則：
    - 透過封裝以及visibility的控制，謹慎的考慮該使用public, protected, private 還是internal的能見度
- 說明：
    - 以這例子，當系統發生異常需要發送通知，假如這通知有Email、簡訊，就應該包裝一個方法呼叫

    
**Bad:**

```csharp
public static void Main(string[] args)
{
    //系統發生異常，需要發通知
    var mailService = new new MailService();
    var messageService = new MessageService();
    ANotificationService notificationService = new NotificationService(mailService, messageService);
    
    //通知負責人
    var user = new User();
    notificationService.SendMail(user, "system error");
    notificationService.SendMessage(user, "system error");
}

public abstract class ANotificationService
{
    protected readonly IMessageService MessageService;
    protected readonly IMailService EmailService;

    protected ANotificationService()
    {
    }

    protected ANotificationService(IMailService emailService, IMessageService messageService)
    {
        EmailService = emailService;
        MessageService = messageService;
    }

    public abstract void SendMail(User receiver, string message);
    public abstract void SendMessage(User receiver, string message);
}

public class NotificationService : ANotificationService
{
    public NotificationService(IMailService emailService, IMessageService messageService) 
        : base(emailService, messageService)
    {
    }
    
    public override void SendMail(User receiver, string message)
    {
        //發Mail
    }

    public override void SendMessage(User receiver, string message)
    {
        //發簡訊
    }
}
```

**Good:**

```csharp

public static void Main(string[] args)
{
    //系統發生異常，需要發通知
    var mailService = new new MailService();
    var messageService = new MessageService();
    ANotificationService notificationService = new NotificationService(mailService, messageService);

    //通知負責人
    var user = new User();
    notificationService.NotifyUser(user, "system error");
}

public abstract class ANotificationService
{
    protected readonly IMessageService MessageService;
    protected readonly IMailService EmailService;

    protected ANotificationService()
    {
    }

    protected ANotificationService(IMailService emailService, IMessageService messageService)
    {
        EmailService = emailService;
        MessageService = messageService;
    }

    public abstract void NotifyUser(User user, string message);
}

public class NotificationService : ANotificationService
{
    public NotificationService(IMailService emailService, IMessageService messageService)
        : base(emailService, messageService)
    {
    }

    public override void NotifyUser(User user, string message)
    {
        //發Mail

        //發簡訊
    }
}

```

- 介面隔離原則，簡稱 LSP
    - 原文定義：
        - Clients should not be forced to depend upon interfaces that they do not use.
        - The dependency of one class to another one should depend on the smallest possible interface.
- 簡單來說：
    - class 與外部的關係，只依賴它需要的最小介面，所以介面不能太肥，要細化
    - 但會與單一職責有衝突，同一個職責的方法，應該要放在一起
    - 介面隔離原則，每個 class 實作某個介面，代表會用到該介面所有的方法，如果用不到所有方法，就應該拆分多個介面
- 目的是什麼：
    - 對介面的範圍進行約束
- 原則：
    - 介面盡量小
    - 介面應高內聚
    - 訂製服務
- 說明：
    - 以這例子，僱員在公司需要工作、面試，但只有主管身份才面試，所以應該要拆分成二個介面
    
**Bad:**

```csharp
internal interface IEmployee
{
    void Work();
    void Interview();
}

internal class Manager : IEmployee
{
    public void Work()
    {
        //寫程式
    }

    public void Interview()
    {
        //面試
    }
}

internal class Staff : IEmployee
{
    public void Work()
    {
        //寫程式
    }

    public void Interview()
    {
        //面試
    }
}
```

**Good:**

```csharp
internal interface IEmployee
{
    void Work();
}

internal interface IManager : IEmployee
{
    void Interview();
}

internal class Manager : IManager
{
    public void Work()
    {
        //寫程式
    }

    public void Interview()
    {
        //面試
    }
}

internal class Staff : IEmployee
{
    public void Work()
    {
        //寫程式
    }
}

```

- 依賴反轉原則，簡稱 DIP
    - 原文定義：
        - High-level modules should not depend on low-level modules. Both should depend on abstractions.
        - Abstractions should not depend upon details. Details should depend on abstractions.
- 簡單來說：
    - class 之間的耦合關係，要多墊一層 interface / abstract 隔開
    - 高層要低層方法，就要透過 interface 來描述抽象行為.

- 說明：
    - 以這例子，早上9點公司開始上班，主管和員工都要工作，但如果這時多一位掃地丫姨，就會異動到 Company 的程式
    - 但換個方式，用外部注入可能是一個集合或是一個工廠，這樣就不需要動到原本程式邏輯
    
**Bad:**

```csharp

public abstract class AEmployee
{
    public abstract void Work();
}

internal class Manager : AEmployee
{
    public override void Work()
    {
        //寫程式
        
        //面試
    }
}    
internal class Staff : AEmployee
{
    public override void Work()
    {
        //寫程式
    }
}

public class Company
{
    private readonly AEmployee _manager;
    private readonly AEmployee _staff;

    public Company(AEmployee manager, AEmployee staff)
    {
        _manager = manager;
        _staff = staff;
    }

    public void Working()
    {
        _manager.Work();
        _staff.Work();
    }
}

```

**Good:**

```csharp
public abstract class AEmployee
{
    public abstract void Work();
}

internal class Manager : AEmployee
{
    public override void Work()
    {
        //寫程式
        
        //面試
    }
}    
internal class Staff : AEmployee
{
    public override void Work()
    {
        //寫程式
    }
}

public class Company
{
    private readonly IEnumerable<AEmployee> _employees;
    
    public Company(IEnumerable<AEmployee> employees)
    {
        _employees = employees;
    }

    public void Working()
    {
        foreach (var employee in _employees)
        {
            employee.Work();
        }
    }
}

```