# SOLID 設計原則

- 單一原則，簡稱 SRP
    - 原文定義：There should never be more than one reason for a class to change
- 簡單來說：
    - 每個 Class 只負責一個職責內容
    - 如果該 Class 擁有多個職責，就應該把每個職責獨立成一個 Class，再以組合方式成為一個 Class
- 好處：
    - Class 複雜度降底，清楚明白職責內容
    - 可讀性、可維護性提高
    - 當需求變更，引起的風險降低
- 舉例：
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