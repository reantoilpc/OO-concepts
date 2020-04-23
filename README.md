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
    - 以這例子，無法實作父類的方法，如果要達成目地，就需要改動父類別程式。

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