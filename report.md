---
title: "Pacific Tours CCSE-CW1 Report"
author: "2242090"
bibliography: references.bib
toc: true
toc-title: Table of Contents
toc-depth: 3
geometry: "left=1.25cm, right=1.25cm, top=1.25cm, bottom=1.25cm, landscape"
csl: harvard-imperial-college-london.csl
---

# 1 Introduction

This report documents the design, development, and implementation of a web-based reservation system for the company, Pacific Tours. The project aims to create an online system that enables customers to search, select, and reserve various hotel accommodations and tour packages that are offered by the company.

The main requirements for this system are as follows:

- Customer registration and login
- Searching and booking hotels
- Searching and booking tours
- Searching and booking packages (consisting of hotel + tour)
- Offering discounts based on booked packages
- Managing booking cancellations and modifications
- Generating booking summaries and availability reports for managers

# 2 System design and development approach

To meet the outlined requirements, a software development methodology, specifically agile, was incorporated to iteratively gather requirements, and design considerations to implement and demonstrate credible working progression within set cycles.

## 2.1 Agile software development methodology

The iterative approach to developing software has been highly beneficial to programmers to improve their skills and organisations in estimating the necessary timespan required for certain tasks [@edeki2015]. Its traits of flexibility, a clear-defined scope of requirements, quick adaptability as well and pragmatism to deliverables make it suitable for software companies to survive within evolving landscapes [@brush2022] – whilst maintaining the company's business interests.

Agile programming utilises the idea of sprints, which are iterative repetitions whereby a functionality is taken and developed to produce small new increments. Each sprint follows the typical developmental phases, as seen in more traditional software methodologies such as the waterfall method. These phases include requirements, analysis, design, evolution, and delivery @abrahamsson2017].

## 2.2 Agile application and design considerations

For this specific scenario, however, an agile methodology was adopted with sprints each lasting 2-weeks. This timeframe proved to be adequate for implementing (which arguably took most of the time), demonstrating, and conducting robust testing. Rather than delivering all the requirements at once, it was beneficial to abstract and break the overall scenario down into key components and decompose further if required.

Throughout, the goal was to promote code that is maintainable, extendable, and also secure. This can be achieved through best coding practices such as:

- Using dependency injection for loose coupling.
- Implementing input validation for all forms.
- Ensuring database parameters are properly formatted, i.e. sanitised.

The sprints were broken down into the following as so, assuming that the project took nearly 2-months. Furthermore, a Kanban-style Jira board provided more of a visual representations in the form of tickets to complete, as opposed to the tabular format presented below.

### 2.2.1 Sprint 1

| Tasks                                                                        | Interpretation                                                                                                            |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Choosing suitable web technologies                                           | ASP.NET C# with both client and server side combined                                                                      |
| Setting up robust development environment                                    | Adequate system resources for high performance, Windows 10 OS, Microsoft SQL Management Studio, stable network connection |
| Familiarising with technologies by developing small Proof of Concepts (POCs) | Experiment with Blazor WebAssembly, ASP.NET default scaffolded classes etc.                                               |
| Deciding the project structure and architecture                              | Single ASP.NET Core Web application with folders for Pages, Services, Exceptions, Models                                  |

### 2.2.2 Sprint 2

| Tasks                              | Interpretation                                                                                  |
| ---------------------------------- | ----------------------------------------------------------------------------------------------- |
| Setup version control system       | Using a private GitHub repository                                                               |
| Configure SQL database settings    | Using Microsoft SQL Management Studio, database context class, Entity Framework Core Migrations |
| Integrate user model               | Using Entity Framework Core Identity, ApplicationUser model                                     |
| Add registration, login and logout | Using ASP.NET scaffolding for default registration, login and logout pages and functionality    |

### 2.2.3 Sprint 3

| Tasks                                                                 | Interpretation                                                                                               |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Decide on database entities                                           | Such Hotels, Tours, Availabilities, etc.                                                                     |
| Discern entity relations between tables                               | Mostly one-to-many/many-to-one with a few one-to-one relations                                               |
| Test SQL database and interact with it via code                       | Use Entity Framework Core Migrations to update database and use data context class to facilitate interaction |
| Initially draft the UI for any required pages and their functionality | Such as Bookings, Edit Bookings, View Bookings and so on                                                     |

### 2.2.4 Sprint 4

| Tasks                                      | Interpretation                                                                                                |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| Implement Bookings page UI                 | Tab based UI for Hotels, Tours and Packages and relevant form fields                                          |
| Implement Bookings functionality           | Configure database query and write to database                                                                |
| Implement View Bookings page UI            | Table based UI for Hotels, Tours and Packages with relevant details plus Edit and Cancel buttons              |
| Implement View Bookings functionality      | Hotels, Tours or Packages booked should be rendered to the page dynamically based on database state           |
| Implement Edit Bookings page UI            | Derive from Bookings page form UI for each Hotels, Tours and Packages                                         |
| Implement Edit Bookings page functionality | Redirect to relevant edit page on button click, search based on dates, update database and View Bookings page |

# 3 System functionality and features

## 3.1 Key requirement 1: Customer account management

This requirement encompasses user login, registration, logout, and their relevant UI counterparts. Mainly through ASP.NET Identity dependency and scaffolding, it provided a customisable foundation for which scenario-specific features can be implemented with ease.

First, looking at the `ApplicationUser` model:

```csharp
using Microsoft.AspNetCoreIdentity

namespace PacificTours.Models
{
    public class ApplicationUser : IdentityUser
    {
        public String FirstName { get; set; } = "";
        public String LastName { get; set; } = "";
        public String Address { get; set; } = "";
        public String PassportNumber { get; set; } = "";
        public DateTime CreatedAt { get; set; }
    }
}
```

This extends ASP.NET's in-built `IdentityUser`, meaning that additional properties vital to a user model, such as `PasswordHash`, `UserName`, `Email`, and `PhoneNumber` are already a part of that class by default and do not need to be redefined.

Next, using ASP.NET's scaffolding, automatic files for Login (`Login.cshtml` and `Login.cshtml.cs`), files for Registration (`Registration.cshtml` and `Registration.cshtml.cs`) as well as Logout (`Logout.cshtml` and `Logout.cshtml.cs`) are generated with their relevant logic in their `OnPostAsync` methods. Then based on the `.cshtml` form submit, the `OnPostAsync` function, checks the user using `SignInManager` class (for Login and Logout) and `UserManager` class (for Registration). For example, looking at the `Login.cshtml.cs`'s `OnPostAsync` method, first checks if the `ModelState` is valid. If so, it attempts to get the user using `PasswordSignInAsync` to find the user from the database and uses the await keyword since the function is of type `async`. Based on that, it informs the UI with the relevant messages as well as redirection to the appropriate pages.

```csharp
public async Task<IActionResult> OnPostAsync(string returnUrl = null)
        {
            returnUrl ??= Url.Content("~/");

            ExternalLogins = (await _signInManager.GetExternalAuthenticationSchemesAsync()).ToList();

            if (ModelState.IsValid)
            {
                var result = await _signInManager.PasswordSignInAsync(Input.Email, Input.Password, Input.RememberMe, lockoutOnFailure: false);
                if (result.Succeeded)
                {
                    _logger.LogInformation("User logged in.");
                    return LocalRedirect(returnUrl);
                }
                if (result.RequiresTwoFactor)
                {
                    return RedirectToPage("./LoginWith2fa", new { ReturnUrl = returnUrl, RememberMe = Input.RememberMe });
                }
                if (result.IsLockedOut)
                {
                    _logger.LogWarning("User account locked out.");
                    return RedirectToPage("./Lockout");
                }
                else
                {
                    ModelState.AddModelError(string.Empty, "Invalid login attempt.");
                    return Page();
                }
            }
```

In general, since the logic for these is repetitive and implemented by default via scaffolding, they do not need to be explored in depth. An abstract explanation of these classes first defines the bind properties that link to the elements in the relevant `.cshtml` page. For example, in the `Login.cshtml.cs` file, the `InputModel` class defines these properties and can bind to the `Login.cshtml` page via the `[BindProperty]` data annotation. Furthermore, the annotations for the properties in the `InputModel` class serve as input validation.

```csharp
[BindProperty]
public InputModel Input { get; set; }

public class InputModel
        {
            [Required]
            [EmailAddress]
            public string Email { get; set; }

            [Required]
            [DataType(DataType.Password)]
            public string Password { get; set; }

            [Display(Name = "Remember me?")]
            public bool RememberMe { get; set; }
        }
```

For more detail see Appendix [5.2](#52-scaffolding-generated-files-for-user-account-management) for the content of all files relating to user account management.

# 4 Cyber security implementation

# 5 Appendices

## 5.1 Link to GitHub repository

[Pacific Tours (pacific-tours-ccse-cw1)](https://github.com/iArcanic/pacific-tours-ccse-cw1)

## 5.2 Scaffolding generated files for user account management

### 5.2.1 [`Login.cshtml`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Areas/Identity/Pages/Account/Login.cshtml)

```csharp
@page
@model LoginModel

@{
    ViewData["Title"] = "Log in";
}

<div class="row">
    <div class="col-md-4 mx-auto rounded border p-3">
        <section>
            <h2 class="text-center mb-3">Please log in.</h2>
            <hr />
            <form id="account" method="post">
                <div asp-validation-summary="ModelOnly" class="text-danger" role="alert"></div>
                <div class="mb-3">
                    <label class="form-label">Email</label>
                    <input asp-for="Input.Email" class="form-control" value="scootham@domain.com" />
                    <span asp-validation-for="Input.Email" class="text-danger"></span>
                </div>
                <div class="mb-3">
                    <label class="form-label">Password</label>
                    <input asp-for="Input.Password" class="form-control" value="Android123!"/>
                    <span asp-validation-for="Input.Password" class="text-danger"></span>
                </div>
                <div class="checkbox mb-3">
                    <label asp-for="Input.RememberMe" class="form-label">
                        <input class="form-check-input" asp-for="Input.RememberMe" />
                        @Html.DisplayNameFor(m => m.Input.RememberMe)
                    </label>
                </div>
                <div class="row mb-3">
                    <div class="col d-grid">
                        <button type="submit" class="btn btn-primary">Log in</button>
                    </div>
                    <div class="col d-grid">
                        <a class="btn btn-outline-primary" href="/" role="button">Cancel</a>
                    </div>
                </div>
                <div>
                    <a class="btn btn-link" id="forget-password" asp-page="./ForgotPassword">Forget password</a>
                </div>
            </form>
        </section>
    </div>
</div>
```

### 5.2.2 [`Login.cshtml.cs`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Areas/Identity/Pages/Account/Login.cshtml.cs)

```csharp
// Licensed to the .NET Foundation under one or more agreements.
// The .NET Foundation licenses this file to you under the MIT license.
#nullable disable

using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using asp_net_core_web_app_authentication_authorisation.Models;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Identity.UI.Services;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Logging;

namespace asp_net_core_web_app_authentication_authorisation.Areas.Identity.Pages.Account
{
    public class LoginModel : PageModel
    {
        private readonly SignInManager<ApplicationUser> _signInManager;
        private readonly ILogger<LoginModel> _logger;

        public LoginModel(SignInManager<ApplicationUser> signInManager, ILogger<LoginModel> logger)
        {
            _signInManager = signInManager;
            _logger = logger;
        }

        /// <summary>
        ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
        ///     directly from your code. This API may change or be removed in future releases.
        /// </summary>
        [BindProperty]
        public InputModel Input { get; set; }

        /// <summary>
        ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
        ///     directly from your code. This API may change or be removed in future releases.
        /// </summary>
        public IList<AuthenticationScheme> ExternalLogins { get; set; }

        /// <summary>
        ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
        ///     directly from your code. This API may change or be removed in future releases.
        /// </summary>
        public string ReturnUrl { get; set; }

        /// <summary>
        ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
        ///     directly from your code. This API may change or be removed in future releases.
        /// </summary>
        [TempData]
        public string ErrorMessage { get; set; }

        /// <summary>
        ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
        ///     directly from your code. This API may change or be removed in future releases.
        /// </summary>
        public class InputModel
        {
            /// <summary>
            ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
            ///     directly from your code. This API may change or be removed in future releases.
            /// </summary>
            [Required]
            [EmailAddress]
            public string Email { get; set; }

            /// <summary>
            ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
            ///     directly from your code. This API may change or be removed in future releases.
            /// </summary>
            [Required]
            [DataType(DataType.Password)]
            public string Password { get; set; }

            /// <summary>
            ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
            ///     directly from your code. This API may change or be removed in future releases.
            /// </summary>
            [Display(Name = "Remember me?")]
            public bool RememberMe { get; set; }
        }

        public async Task OnGetAsync(string returnUrl = null)
        {
            if (!string.IsNullOrEmpty(ErrorMessage))
            {
                ModelState.AddModelError(string.Empty, ErrorMessage);
            }

            returnUrl ??= Url.Content("~/");

            // Clear the existing external cookie to ensure a clean login process
            await HttpContext.SignOutAsync(IdentityConstants.ExternalScheme);

            ExternalLogins = (await _signInManager.GetExternalAuthenticationSchemesAsync()).ToList();

            ReturnUrl = returnUrl;
        }

        public async Task<IActionResult> OnPostAsync(string returnUrl = null)
        {
            returnUrl ??= Url.Content("~/");

            ExternalLogins = (await _signInManager.GetExternalAuthenticationSchemesAsync()).ToList();

            if (ModelState.IsValid)
            {
                // This doesn't count login failures towards account lockout
                // To enable password failures to trigger account lockout, set lockoutOnFailure: true
                var result = await _signInManager.PasswordSignInAsync(Input.Email, Input.Password, Input.RememberMe, lockoutOnFailure: false);
                if (result.Succeeded)
                {
                    _logger.LogInformation("User logged in.");
                    return LocalRedirect(returnUrl);
                }
                if (result.RequiresTwoFactor)
                {
                    return RedirectToPage("./LoginWith2fa", new { ReturnUrl = returnUrl, RememberMe = Input.RememberMe });
                }
                if (result.IsLockedOut)
                {
                    _logger.LogWarning("User account locked out.");
                    return RedirectToPage("./Lockout");
                }
                else
                {
                    ModelState.AddModelError(string.Empty, "Invalid login attempt.");
                    return Page();
                }
            }

            // If we got this far, something failed, redisplay form
            return Page();
        }
    }
}
```

### 5.2.3 [`Register.cshtml`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Areas/Identity/Pages/Account/Register.cshtml)

```csharp
@page
@model RegisterModel
@{
    ViewData["Title"] = "Register";
}

<div class="row">
    <div class="col-lg-6 mx-auto rounded border p-3">
        <h2>Create a new account.</h2>
        <hr />
        <div asp-validation-summary="ModelOnly" class="text-danger" role="alert"></div>
        <form id="registerForm" asp-route-returnUrl="@Model.ReturnUrl" method="post">
            <div class="row mb-3">
                <label class="col-sm-4 col-form-label">First Name</label>
                <div class="col-sm-8">
                    <input class="form-control" asp-for="Input.FirstName" />
                    <span asp-validation-for="Input.FirstName" class="text-danger"></span>
                </div>
            </div>
            <div class="row mb-3">
                <label class="col-sm-4 col-form-label">Last Name</label>
                <div class="col-sm-8">
                    <input class="form-control" asp-for="Input.LastName" />
                    <span asp-validation-for="Input.LastName" class="text-danger"></span>
                </div>
            </div>
            <div class="row mb-3">
                <label class="col-sm-4 col-form-label">Email</label>
                <div class="col-sm-8">
                    <input class="form-control" asp-for="Input.Email"/>
                    <span asp-validation-for="Input.Email" class="text-danger"></span>
                </div>
            </div>
            <div class="row mb-3">
                <label class="col-sm-4 col-form-label">Password</label>
                <div class="col-sm-8">
                    <input class="form-control" asp-for="Input.Password" />
                    <span asp-validation-for="Input.Password" class="text-danger"></span>
                </div>
            </div>
            <div class="row mb-3">
                <label class="col-sm-4 col-form-label">Confirm Password</label>
                <div class="col-sm-8">
                    <input class="form-control" asp-for="Input.ConfirmPassword" />
                    <span asp-validation-for="Input.ConfirmPassword" class="text-danger"></span>
                </div>
            </div>
            <div class="row mb-3">
                <label class="col-sm-4 col-form-label">Phone number</label>
                <div class="col-sm-8">
                    <input class="form-control" asp-for="Input.PhoneNumber" />
                    <span asp-validation-for="Input.PhoneNumber" class="text-danger"></span>
                </div>
            </div>
            <div class="row mb-3">
                <label class="col-sm-4 col-form-label">Address</label>
                <div class="col-sm-8">
                    <input class="form-control" asp-for="Input.Address" />
                    <span asp-validation-for="Input.Address" class="text-danger"></span>
                </div>
            </div>
            <div class="row mb-3">
                <label class="col-sm-4 col-form-label">Passport number</label>
                <div class="col-sm-8">
                    <input class="form-control" asp-for="Input.PassportNumber" />
                    <span asp-validation-for="Input.PassportNumber" class="text-danger"></span>
                </div>
            </div>
            <div class="row mb-3">
                <div class="offset-sm-4 col-sm-4 d-grid">
                    <button type="submit" class="btn btn-primary">Register</button>
                </div>
                <div class="colcol-sm-4 d-grid">
                    <a class="btn btn-outline-primary" href="/" role="button">Cancel</a>
                </div>
            </div>
        </form>
    </div>
</div>
```

### 5.2.4 [`Register.cshtml.cs`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Areas/Identity/Pages/Account/Register.cshtml.cs)

```csharp
// Licensed to the .NET Foundation under one or more agreements.
// The .NET Foundation licenses this file to you under the MIT license.
#nullable disable

using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using System.Text;
using System.Text.Encodings.Web;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authorization;
using asp_net_core_web_app_authentication_authorisation.Models;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Identity.UI.Services;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.AspNetCore.WebUtilities;
using Microsoft.Extensions.Logging;

namespace asp_net_core_web_app_authentication_authorisation.Areas.Identity.Pages.Account
{
    public class RegisterModel : PageModel
    {
        private readonly SignInManager<ApplicationUser> _signInManager;
        private readonly UserManager<ApplicationUser> _userManager;
        private readonly IUserStore<ApplicationUser> _userStore;
        private readonly IUserEmailStore<ApplicationUser> _emailStore;
        private readonly ILogger<RegisterModel> _logger;
        private readonly IEmailSender _emailSender;

        public RegisterModel(
            UserManager<ApplicationUser> userManager,
            IUserStore<ApplicationUser> userStore,
            SignInManager<ApplicationUser> signInManager,
            ILogger<RegisterModel> logger,
            IEmailSender emailSender)
        {
            _userManager = userManager;
            _userStore = userStore;
            _emailStore = GetEmailStore();
            _signInManager = signInManager;
            _logger = logger;
            _emailSender = emailSender;
        }

        /// <summary>
        ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
        ///     directly from your code. This API may change or be removed in future releases.
        /// </summary>
        [BindProperty]
        public InputModel Input { get; set; }

        /// <summary>
        ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
        ///     directly from your code. This API may change or be removed in future releases.
        /// </summary>
        public string ReturnUrl { get; set; }

        /// <summary>
        ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
        ///     directly from your code. This API may change or be removed in future releases.
        /// </summary>
        public IList<AuthenticationScheme> ExternalLogins { get; set; }

        /// <summary>
        ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
        ///     directly from your code. This API may change or be removed in future releases.
        /// </summary>
        public class InputModel
        {
            [Required]
            public string FirstName { get; set; }

            [Required]
            public string LastName { get; set; }

            [Required]
            public string PhoneNumber { get; set; }

            [Required]
            public string Address { get; set; }

            [Required]
            public string PassportNumber { get; set; }

            /// <summary>
            ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
            ///     directly from your code. This API may change or be removed in future releases.
            /// </summary>
            [Required]
            [EmailAddress]
            [Display(Name = "Email")]
            public string Email { get; set; }

            /// <summary>
            ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
            ///     directly from your code. This API may change or be removed in future releases.
            /// </summary>
            [Required]
            [StringLength(100, ErrorMessage = "The {0} must be at least {2} and at max {1} characters long.", MinimumLength = 6)]
            [DataType(DataType.Password)]
            [Display(Name = "Password")]
            public string Password { get; set; }

            /// <summary>
            ///     This API supports the ASP.NET Core Identity default UI infrastructure and is not intended to be used
            ///     directly from your code. This API may change or be removed in future releases.
            /// </summary>
            [DataType(DataType.Password)]
            [Display(Name = "Confirm password")]
            [Compare("Password", ErrorMessage = "The password and confirmation password do not match.")]
            public string ConfirmPassword { get; set; }
        }


        public async Task OnGetAsync(string returnUrl = null)
        {
            ReturnUrl = returnUrl;
            ExternalLogins = (await _signInManager.GetExternalAuthenticationSchemesAsync()).ToList();
        }

        public async Task<IActionResult> OnPostAsync(string returnUrl = null)
        {
            returnUrl ??= Url.Content("~/");
            ExternalLogins = (await _signInManager.GetExternalAuthenticationSchemesAsync()).ToList();
            if (ModelState.IsValid)
            {
                var user = new ApplicationUser()
                {
                    FirstName = Input.FirstName,
                    LastName = Input.LastName,
                    UserName = Input.Email,
                    Email = Input.Email,
                    PhoneNumber = Input.PhoneNumber,
                    Address = Input.Address,
                    CreatedAt = DateTime.Now,
                    PassportNumber = Input.PassportNumber
                };

                var result = await _userManager.CreateAsync(user, Input.Password);

                if (result.Succeeded)
                {
                    _logger.LogInformation("User created a new account with password.");

                    var userId = await _userManager.GetUserIdAsync(user);
                    var code = await _userManager.GenerateEmailConfirmationTokenAsync(user);
                    code = WebEncoders.Base64UrlEncode(Encoding.UTF8.GetBytes(code));
                    var callbackUrl = Url.Page(
                        "/Account/ConfirmEmail",
                        pageHandler: null,
                        values: new { area = "Identity", userId = userId, code = code, returnUrl = returnUrl },
                        protocol: Request.Scheme);

                    await _emailSender.SendEmailAsync(Input.Email, "Confirm your email",
                        $"Please confirm your account by <a href='{HtmlEncoder.Default.Encode(callbackUrl)}'>clicking here</a>.");

                    if (_userManager.Options.SignIn.RequireConfirmedAccount)
                    {
                        return RedirectToPage("RegisterConfirmation", new { email = Input.Email, returnUrl = returnUrl });
                    }
                    else
                    {
                        await _signInManager.SignInAsync(user, isPersistent: false);
                        return LocalRedirect(returnUrl);
                    }
                }
                foreach (var error in result.Errors)
                {
                    ModelState.AddModelError(string.Empty, error.Description);
                }
            }

            // If we got this far, something failed, redisplay form
            return Page();
        }

        private ApplicationUser CreateUser()
        {
            try
            {
                return Activator.CreateInstance<ApplicationUser>();
            }
            catch
            {
                throw new InvalidOperationException($"Can't create an instance of '{nameof(ApplicationUser)}'. " +
                    $"Ensure that '{nameof(ApplicationUser)}' is not an abstract class and has a parameterless constructor, or alternatively " +
                    $"override the register page in /Areas/Identity/Pages/Account/Register.cshtml");
            }
        }

        private IUserEmailStore<ApplicationUser> GetEmailStore()
        {
            if (!_userManager.SupportsUserEmail)
            {
                throw new NotSupportedException("The default UI requires a user store with email support.");
            }
            return (IUserEmailStore<ApplicationUser>)_userStore;
        }
    }
}
```

### 5.2.5 [`Logout.cshtml`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Areas/Identity/Pages/Account/Logout.cshtml)

```csharp
@page
@model LogoutModel
@{
    ViewData["Title"] = "Log out";
}

<header>
    <h1>@ViewData["Title"]</h1>
    @{
        if (User.Identity?.IsAuthenticated ?? false)
        {
            <form class="form-inline" asp-area="Identity" asp-page="/Account/Logout" asp-route-returnUrl="@Url.Page("/", new { area = "" })" method="post">
                <button type="submit" class="nav-link btn btn-link text-dark">Click here to Logout</button>
            </form>
        }
        else
        {
            <p>You have successfully logged out of the application.</p>
        }
    }
</header>
```

### 5.2.6 [`Logout.cshtml`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Areas/Identity/Pages/Account/Logout.cshtml.cs)

```csharp
// Licensed to the .NET Foundation under one or more agreements.
// The .NET Foundation licenses this file to you under the MIT license.
#nullable disable

using System;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using asp_net_core_web_app_authentication_authorisation.Models;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Logging;

namespace asp_net_core_web_app_authentication_authorisation.Areas.Identity.Pages.Account
{
    public class LogoutModel : PageModel
    {
        private readonly SignInManager<ApplicationUser> _signInManager;
        private readonly ILogger<LogoutModel> _logger;

        public LogoutModel(SignInManager<ApplicationUser> signInManager, ILogger<LogoutModel> logger)
        {
            _signInManager = signInManager;
            _logger = logger;
        }

        public async Task<IActionResult> OnPost(string returnUrl = null)
        {
            await _signInManager.SignOutAsync();
            _logger.LogInformation("User logged out.");
            if (returnUrl != null)
            {
                return LocalRedirect(returnUrl);
            }
            else
            {
                // This needs to be a redirect so that the browser performs a new
                // request and the identity for the user gets updated.
                return RedirectToPage();
            }
        }
    }
}
```

# 6 References
