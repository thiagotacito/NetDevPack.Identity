<img src="https://repository-images.githubusercontent.com/268701472/8bf84980-a6ce-11ea-83da-e2133c5a3a7a" alt=".NET DevPack" width="300px" />

What is the .NET DevPack.Identity?
=====================
.NET DevPack Identity is a set of common implementations to help you implementing ASP.NET Identity, JWT, claims validation and another facilities

## Give a Star! :star:
If you liked the project or if NetDevPack helped you, please give a star ;)

## Get Started

| Package |  Version | Popularity |
| ------- | ----- | ----- |
| `NetDevPack.Identity` | [![NuGet](https://img.shields.io/nuget/v/NetDevPack.Identity.svg)](https://nuget.org/packages/NetDevPack.Identity) | [![Nuget](https://img.shields.io/nuget/dt/NetDevPack.Identity.svg)](https://nuget.org/packages/NetDevPack.Identity) |

.NET DevPack.Identity can be installed in your ASP.NET Core application using the Nuget package manager or the `dotnet` CLI.
```
dotnet add package NetDevPack.Identity
```
If you want to use our IdentityDbContext (ASP.NET Identity standard) you will need to create the Identity tables. Set your connection string in the `appsettings.json` and follow the next steps:

Add the IdentityDbContext configuration in your `startup.cs`:
```csharp
services.AddIdentityEntityFrameworkContextConfiguration(Configuration, GetType().Namespace,"DefaultConnection");
```

>**Note:** You need to inform the namespace to avoid migration errors
>
>**Note:** If you don't inform the connection string name the value adopted will be _IdentityConnection_

Add the Identity configuration in `ConfigureServices` method of your `startup.cs`:
```csharp
services.AddIdentityConfiguration();
```
>**Note:** This extension returns an IdentityBuilder to allow you extending the configuration

Add the Identity configuration in `ConfigureServices` method of your `startup.cs`:
```csharp
services.AddIdentityConfiguration();
```
Add the Identity configuration in `Configure` method of your `startup.cs`:
```csharp
app.UseAuthConfiguration();
```
>**Note:** This method need to be set between `app.UseRouting()` and `app.UseEndpoints()`

Run the command to generate the migration files:
```
dotnet ef migrations add Initial --project <Your patch>/<Your Project>.csproj
```

Run the command to generate the database:
```
dotnet ef database update --project <Your patch>/<Your Project>.csproj
```

After execute this steps you will be all set to use the Identity in your Application.

### Configuring JWT
If you want to generate JSON Web Tokens in your application you need to add the JWT configuration in `ConfigureServices` method of your `startup.cs` like the sample below:

```csharp
services.AddIdentityEntityFrameworkContextConfiguration(Configuration, GetType().Namespace,"DefaultConnection");
services.AddJwtConfiguration(Configuration, "AppSettings"); // <=== HERE
services.AddIdentityConfiguration();
```
>**Note:** If you don't inform the configuration name the value adopted will be _AppJwtSettings_

Set your `appsettings.json` file with this values:
```json
"AppSettings": {
    "SecretKey": "MYSECRETSUPERSECRET",
    "Expiration": 2,
    "Issuer": "SampleApp",
	"Audience": "https://localhost"
}
``` 
|Key|Meaning|
|--|--|
|SecretKey  | Is your key to build JWT. This value need to be stored in a safe place in the production way |
|Expiration| Expiration time in hours  |
|Issuer| The name of the JWT issuer  |
|Audience| The domain that the JWT will be valid. Can be a string collection  |

### Generating JWT
You will need to set some dependencies in your Authentication Controller:

```csharp
private readonly SignInManager<IdentityUser> _signInManager;
private readonly UserManager<IdentityUser> _userManager;
private readonly AppJwtSettings _appJwtSettings;

public AuthController(SignInManager<IdentityUser> signInManager,
				      UserManager<IdentityUser> userManager,
					  IOptions<AppJwtSettings> appJwtSettings)
{
    _signInManager = signInManager;
    _userManager = userManager;
    _appJwtSettings = appJwtSettings.Value;
}
```
>**Note:** The _AppJwtSettings_ is our dependency and is configured internally during JWT setup (in `startup.cs` file). You just need to inject it in your controller.
>
>**Note:** The _SignInManager_ and _UserManager_ classes is native from Identity and provided in NetDevPack.Identity. You just need to inject it in your controller.

After user register or login process you can generate a JWT to respond the request. Use our implementation, you just need inform the user email and the dependencies injected in your controller:

```csharp
return new JwtBuilder()
	.WithUserManager(_userManager)
	.WithJwtSettings(_appJwtSettings)
	.WithEmail(email)
	.BuildToken();
```
>**Note:** This builder can return a single string with JWT or a complex object `UserResponse` if you want return more data than a single JWT string.

#### Adding Claims to your JWT
You can call more methods in `JwtBuilder` to provide more information about the user:
```csharp
return new JwtBuilder()
    .WithUserManager(_userManager)
    .WithJwtSettings(_appJwtSettings)
    .WithEmail(email)
    .WithJwtClaims()
    .WithUserClaims()
    .WithUserRoles()
    .BuildToken();
```
|Method|Meaning|
|--|--|
|WithJwtClaims()| Claims of JWT like `sub`, `jti`, `nbf` and others |
|WithUserClaims()| The user claims registered in `AspNetUserClaims` table|
|WithUserRoles()| The user roles (as claims) registered in `AspNetUserRoles` table  |
|BuildToken()| Build and return the JWT as single string  |

If you want return your complex object `UserResponse` you need to change the last method to:
```csharp
return new JwtBuilder()
    .WithUserManager(_userManager)
    .WithJwtSettings(_appJwtSettings)
    .WithEmail(email)
    .WithJwtClaims()
    .WithUserClaims()
    .WithUserRoles()
    .BuildUserResponse();
```

## Examples
Use the [sample application](https://github.com/NetDevPack/NetDevPack.Identity/tree/master/src/Samples/AspNetCore.Jwt.Sample) to understand how NetDevPack.Identity can be implemented and help you to decrease the complexity of your application and development time.

## About
.NET DevPack was developed by [Eduardo Pires](http://eduardopires.net.br) under the [MIT license](LICENSE).
