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