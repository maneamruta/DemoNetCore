# DemoNetCore

## 1. Create web project 
   1. Open Visual Studio and goto 
   1. File Menu -> Web Application (Model - View - Controller)
   
## 2. Restore and Build
   1. Right click on solution in solution explorer and select Restore nuget packages 
   2. Right click on solution in solution explorer and Build solution

## 3. Run solution
* Click the play button on visual studio -> should open a homepage for current solution with Home and Privacy page

## 4. Install Identity library to project 
* Right click on project -> Manage nuget packages -> search Microsoft.Extensions.Identity.Core -> select and Add package

## 5. Register Identity library 
* Goto project startup class -> Configure services method -> Add service as below 
```C#
services.AddIdentityCore<string>(options => { });
```

## 6. Add new user class 
<details> 
  <summary>New user class</summary>

```C#
    public class DemoUser
    {
        
        public string Id { get; set; }
        public string UserName { get; set; }
        public string NormalizedUserName { get; set; }
        public string PasswordHash { get; set; }
    }
```
</details> 

## 7. Add user store 
<details> 
  <summary>Add new class DemoUserStore which implements IUserStore<DemoUser> and IUserPasswordStore </summary>

```C#
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Identity;

namespace DemoNetCore
{
    public class DemoUserStore : IUserStore<DemoUser>, IUserPasswordStore<DemoUser>
    {
        private List<DemoUser> users = new List<DemoUser>();

        public DemoUserStore()
        {
        }

        public Task<IdentityResult> CreateAsync(DemoUser user, CancellationToken cancellationToken)
        {
            users.Add(user);
            return Task.FromResult(IdentityResult.Success);
        }

        public Task<IdentityResult> DeleteAsync(DemoUser user, CancellationToken cancellationToken)
        {
            users.Remove(user);
            return Task.FromResult(IdentityResult.Success);
        }

        public void Dispose()
        { 
        }

        public Task<DemoUser> FindByIdAsync(string userId, CancellationToken cancellationToken)
        {
            return Task.FromResult(users.FirstOrDefault(x => x.Id == userId));
        }

        public Task<DemoUser> FindByNameAsync(string normalizedUserName, CancellationToken cancellationToken)
        {
            return Task.FromResult(users.FirstOrDefault(x => x.NormalizedUserName == normalizedUserName));
        }

        public Task<string> GetNormalizedUserNameAsync(DemoUser user, CancellationToken cancellationToken)
        {
            return Task.FromResult(user.NormalizedUserName);
        }

        public Task<string> GetPasswordHashAsync(DemoUser user, CancellationToken cancellationToken)
        {
            return Task.FromResult(user.PasswordHash);
        }

        public Task<string> GetUserIdAsync(DemoUser user, CancellationToken cancellationToken)
        {
            return Task.FromResult(user.Id);
        }

        public Task<string> GetUserNameAsync(DemoUser user, CancellationToken cancellationToken)
        {
            return Task.FromResult(user.UserName);
        }

        public Task<bool> HasPasswordAsync(DemoUser user, CancellationToken cancellationToken)
        {
            return Task.FromResult(user.PasswordHash != null);
        }

        public Task SetNormalizedUserNameAsync(DemoUser user, string normalizedName, CancellationToken cancellationToken)
        {
            return Task.FromResult(user.NormalizedUserName = normalizedName);
        }

        public Task SetPasswordHashAsync(DemoUser user, string passwordHash, CancellationToken cancellationToken)
        {
            return Task.FromResult(user.PasswordHash = passwordHash);
        }

        public Task SetUserNameAsync(DemoUser user, string userName, CancellationToken cancellationToken)
        {
            return Task.FromResult(user.UserName = userName);
        }

        public Task<IdentityResult> UpdateAsync(DemoUser user, CancellationToken cancellationToken)
        {
            throw new System.NotImplementedException();
        }
    }
}

```
</details>

## 8. Update StartUp class to use user store created in earlier step

```C#
  services.AddIdentityCore<DemoUser>(options => { });
  services.AddSingleton<IUserStore<DemoUser>, DemoUserStore>();
```

## 9. Update HomeController to have Register methods
<details> 
  <summary>Register get/post methods </summary>
   
```C#
        [HttpGet]
        public IActionResult Register()
        {
            return View();
        }

        [HttpPost]
        public async Task<IActionResult> Register(RegisterModel registerModel)
        {

        }
```
</details>

## 10. Add Register Model
<details> 
  <summary>Register Model </summary>

```C#
    public class RegisterModel
    {
       public string UserName { get; set; }

       [DataType(DataType.Password)]
       public string Password { get; set; }

        [Compare("Password")]
        [DataType(DataType.Password)]
        public string ConfirmPassword { get; set; }
    }
```
</details>

## 11. Add Register view

<details> 
  <summary>Register View </summary>

```HTML
@model RegisterModel

<div class="row">
    <div class="col-sm-8 col-sm-offset-2">
        <h1 class="text-center">Register</h1>

        <form asp-action="Register" method="post" class="form-horizontal">
            <div class="form-group">
                <label asp-for="UserName" class="col-sm-2"></label>
                <div class="col-sm-10">
                    <input asp-for="UserName" class="form-control" />
                </div>
            </div>
            <div class="form-group">
                <label asp-for="Password" class="col-sm-2"></label>
                <div class="col-sm-10">
                    <input asp-for="Password" class="form-control" />
                </div>
            </div>
            <div class="form-group">
                <label asp-for="ConfirmPassword" class="col-sm-2"></label>
                <div class="col-sm-10">
                    <input asp-for="ConfirmPassword" class="form-control" />
                </div>
            </div>
            <div><button type="submit" class="btn btn-primary">Submit</button></div>
            <div asp-validation-summary="All"></div>
        </form>
    </div>
</div> 

```
</details>

## 12. Add success view 
<details> 
  <summary>Success View </summary>

```HTML
<div class="alert alert-success">
    <p>Success!!!</p>
</div>
```
</details>

## 13. Modify Register post method
<details> 
  <summary>Register method content </summary>
   
```C#
            if(ModelState.IsValid)
            {
                var user = await _userManager.FindByNameAsync(registerModel.UserName);

                if(user == null)
                {
                    user = new DemoUser
                    {
                        Id = Guid.NewGuid().ToString(),
                        UserName = registerModel.UserName
                    };

                    var result = await _userManager.CreateAsync(user, registerModel.Password);
                }

                return View("Success");
            }

            return View();  
```

</details>

## 14. Add Login get/post to home controller

<details> 
  <summary>Login get/post methods </summary>

```C#
        [HttpGet]
        public IActionResult Login()
        {
            return View();
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Login(LoginModel loginModel)
        {
            
        }
```
</details> 

## 15. Add Login Model

<details> 
  <summary>Login Model</summary>

```C#
    public class LoginModel
    {
        public string UserName { get; set; }

        [DataType(DataType.Password)]
        public string Password { get; set; }
    }
```
</details>

## 16. Add login view

<details> 
  <summary>Login View </summary>

```HTML
@model LoginModel

<div class="row">
    <div class="col-xs-8 col-xs-offset-2">
        <h1 class="text-center">Login</h1>

        <form asp-action="Login" method="post" class="form-horizontal">
            <div class="form-group">
                <label asp-for="UserName" class="col-sm-2"></label>
                <div class="col-sm-10">
                    <input asp-for="UserName" class="form-control" />
                </div>
            </div>
            <div class="form-group">
                <label asp-for="Password" class="col-sm-2"></label>
                <div class="col-sm-10">
                    <input asp-for="Password" class="form-control" />
                </div>
            </div>
            <button type="submit" class="btn btn-primary">Submit</button>
            <div asp-validation-summary="All"></div>
        </form>
    </div>
</div> 

```
</details>

## 17. Update login post method

<details> 
  <summary>Login post method </summary>

```C#
            if (ModelState.IsValid)
            {
                var user = await _userManager.FindByNameAsync(loginModel.UserName);

                if(user != null && await _userManager.CheckPasswordAsync(user, loginModel.Password))
                {
                    var identity = new ClaimsIdentity("cookies");
                    identity.AddClaim(new Claim(ClaimTypes.NameIdentifier, user.Id));
                    identity.AddClaim(new Claim(ClaimTypes.Name, user.UserName));

                    await HttpContext.SignInAsync("cookies", new ClaimsPrincipal(identity));

                    return RedirectToAction("Index");
                }

                ModelState.AddModelError("", "Invalid username or password");
            }

            return View();

```
</details>

## 18. Configure Login 
  1. Configure service 

```C#
    services.AddAuthentication("cookies").AddCookie("cookies", options => options.LoginPath = "/Home/Login");
```

  2. Configure config

```C#
    app.UseAuthentication();
```
