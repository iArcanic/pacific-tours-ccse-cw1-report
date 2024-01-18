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

## 3.2 Key requirement 2: Bookings

The requirement for this part of the scenario involves a single bookings page in which the user can either book a hotel, tour, or package (a combination of both a hotel and a tour). For this page, inspiration was taken from Bookings.com UI – which features a tab-based selection, where there is a distinct tab for each entity. Therefore, such a format was taken, and the starting code for this was taken from W3Schools [@w3schools2024] (See Appendix [5.3.1](#531-w3schools-tab-based-ui)).

This UI however, has to be adapted for the Bookings system, since the default tab logic doesn't take into account the refresh on form submit, persistence of input box states, additional client-side validation and so on. This whole host of issues involved additional JavaScript client-side validation that cannot be addressed by the server-side C# code alone.

Since the UI and backend logic is virtually similar to each other, looking at the implementation of hotel bookings should suffice.

Starting with the UI, here, towards the top, the `asp-page-handler` attribute of the form element corresponds to `HotelSearch`, i.e. the correct `OnPostAsync` method in the server-side code. This also means that the URL will have a new query parameter, `handler=HotelSearch`, making it easy to distinguish between what tab is selected. Furthermore, there is the necessary validation from the service which is referenced here through the `asp-validation` element tags.

```csharp
<div asp-validation-summary="ModelOnly" class="text-danger" role="alert"></div>
<form id="hotelSearchForm" method="post" asp-page-handler="HotelSearch">
    <div id="Hotels" class="tabcontent">
        <h3>Search Hotels</h3>
        <hr />
        <div class="form-group">
            <label asp-for="HotelSearch.CheckInDate">Select check-in date:</label>
            <input id="hotelCheckInDate" asp-for="HotelSearch.CheckInDate" type="date" class="form-control" />
            <span asp-validation-for="HotelSearch.CheckInDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label asp-for="HotelSearch.CheckInDate">Select check-out date:</label>
            <input id="hotelCheckOutDate" asp-for="HotelSearch.CheckOutDate" type="date" class="form-control" />
            <span asp-validation-for="HotelSearch.CheckOutDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label asp-for="HotelSearch.RoomType">Select room type:</label>
            <select asp-for="HotelSearch.RoomType" asp-items="@Model.HotelSearch.RoomTypes" class="form-control"></select>
            <span asp-validation-for="HotelSearch.RoomType" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label for="hotels">Hotels:</label>
            <select name="hotels" id="hotelsDropdown" onchange='calculateTotalCost("hotelsDropdown", "hotelCheckInDate", "hotelCheckOutDate", "hotelTotalCostMessage")'>
                @foreach (var hotel in Model.HotelSearch.HotelsList)
                {
                    <option value="@hotel.HotelId">@(hotel.Name + " £" + hotel.Cost + " (per night)")</option>
                }
            </select>
            <p id="hotelsDropdownErrorMessage" class="text-danger"></p>
        </div>
        <div class="form-group">
            <input type="submit" name="command" class="btn btn-primary" value="Search" />
        </div>
        <hr />
        <p id="hotelTotalCostMessage">Total Cost: £0</p>
        <div class="form-group">
            <input class="btn btn-primary" name="command" value="Book" onclick="submitHotelSearchForm()" />
        </div>
    </div>
</form>
```

Here below, are the relevant JavaScript functions that are referenced in the code.

As mentioned previously, since the `OnPostAsync` function reloads the page, using the page handler, this `switch` statement checks what form is being submitted and automatically selects the relevant tab as default with the `onClick()` method.

```javascript
let params = new URL(document.location).searchParams;
let handler = params.get("handler");

switch (handler) {
  case "HotelSearch":
    document.getElementById("HotelTab").click();
    break;
  case "TourSearch":
    document.getElementById("TourTab").click();
    break;
  case "PackageBook":
    document.getElementById("PackageTab").click();
    break;
  default:
    break;
}
```

Here, this method checks whether the dropdown box is not empty and displays an appropriate error message by referencing a paragraph tag within the form. It only then submits the form dynamically if the `selectBoxValue` isn't null.

```javascript
function submitHotelSearchForm() {
  var dropdownErrorMessage = document.getElementById(
    "hotelsDropdownErrorMessage"
  );
  dropdownErrorMessage.innerHTML = "";

  var form = document.getElementById("hotelSearchForm");
  var selectBoxValue = form.elements["hotelsDropdown"].value;

  if (!selectBoxValue) {
    dropdownErrorMessage.innerHTML = "Please select a hotel for booking";
    return;
  }

  form.elements["command"].value = "Book";
  form.submit();
}
```

The following series of functions are related to cost calculations for all three tabs.

The first function, `calculateTotal(dropdown, fromDate, toDate)`, extracts the cost value from the dropdown box and calculates the cost of the hotel or tour per night/day by taking the duration of days in between the inputted dates.

```javascript
function calculateTotal(dropdown, fromDate, toDate) {
  var selectedDropdown = document.getElementById(dropdown);
  var selectedOption = selectedDropdown.options[selectedDropdown.selectedIndex];
  var cost = selectedOption.textContent.split(" £")[1].split(" ")[0];

  var startDate = new Date(document.getElementById(fromDate).value);
  var endDate = new Date(document.getElementById(toDate).value);

  var durationInDays = Math.ceil((endDate - startDate) / (1000 * 60 * 60 * 24));

  return parseFloat((cost * durationInDays).toFixed(2));
}

function calculateTotalCost(dropdown, fromDate, toDate, totalCostMessageId) {
  document.getElementById(totalCostMessageId).innerHTML =
    "Total Cost: £" + calculateTotal(dropdown, fromDate, toDate);
}
```

This next function, is specifically for calculating the total cost of a package since it contains both a tour and hotel. Since logic doesn't need to be repeated, it utilises the previous function to calculate the overall cost.

```javascript
function calculateTotalPackageCost() {
  var packageHotelCost = calculateTotal(
    "packageHotelsDropdownId",
    "packageHotelCheckInDate",
    "packageHotelCheckOutDate"
  );
  var packageTourCost = calculateTotal(
    "packageToursDropdownId",
    "packageTourStartDate",
    "packageTourEndDate"
  );

  var packageTotalCost = packageHotelCost + packageTourCost;

  document.getElementById("packageTotalCostMessage").innerHTML =
    "Total Cost: £" + packageTotalCost;
}
```

Finally, upon page reload, an attached event listener upon page reload will check which tab is active and apply the total cost calculation dynamically. This uses the `.className` attribute of the tab switching code.

```javascript
window.addEventListener("load", function () {
  if (document.getElementById("HotelTab").className == "tablinks active") {
    calculateTotalCost(
      "hotelsDropdown",
      "hotelCheckInDate",
      "hotelCheckOutDate",
      "hotelTotalCostMessage"
    );
  } else if (
    document.getElementById("TourTab").className == "tablinks active"
  ) {
    calculateTotalCost(
      "toursDropdown",
      "tourStartDate",
      "tourEndDate",
      "tourTotalCostMessage"
    );
  } else if (
    document.getElementById("PackageTab").className == "tablinks active"
  ) {
    calculateTotalPackageCost();
  }
});
```

Moving onto the server-side function, again, the bind model for the client-side properties are the same. Taking the `OnPostHotelSearchAsync` function as the logic for all three is consistent, it again first checks if the `ModelState`, which is the logic and client-side elements, are valid.

```csharp
public async Task<IActionResult> OnPostHotelSearchAsync(string command, string returnUrl = null)
{
    if (!ModelState.IsValid)
    {
        return Page();
    }
```

Then, it checks if the button clicked is search, via the `command` parameter. This references the `name=command` element for the search button on the client-side. Using the database context, it checks for all available hotels based on the user's dates and returns the found `Hotel` object. It then binds this into the `HotelList` to the client-side element.

```csharp
if (command == "Search")
{
var availableHotels = await _dbContext.HotelAvailabilities
    .Where(ha =>
        ha.AvailableFrom <= HotelSearch.CheckInDate &&
        ha.AvailableTo >= HotelSearch.CheckOutDate &&
        ha.Hotel.RoomType == HotelSearch.RoomType)
    .Select(ha => ha.Hotel)
    .Distinct()
    .ToListAsync();

HotelSearch.HotelsList = availableHotels;

return Page();
```

However, in the else block, i.e. the "Book" button is clicked, a new hotel booking is made, referencing the `HotelBooking` model. This model takes in an unique ID, i.e. the HotelBookingId as a GUID data type, a reference to the `Hotel` which the user selected, the dates and a reference to the current user found via the `IdentityUser`'s `UserManager` class. It is then added to the `HotelBookings` table in the database. Finally, a redirection is made to the "/Payment" page with the booking ID and type of booking (hotel, tour, or package) as a query parameter in the URL. This is then extracted in the payments page for further use.

```csharp
else
{
    var SelectedHotelId = new Guid(Request.Form["hotels"]);
    var CurrentUser = await _userManager.GetUserAsync(User);
    Hotel SelectedHotel = await _dbContext.Hotels.FindAsync(SelectedHotelId);

    var hotelBooking = new HotelBooking
    {
        HotelBookingId = new Guid(),
        HotelId = SelectedHotelId,
        UserId = CurrentUser.Id,
        CheckInDate = HotelSearch.CheckInDate,
        CheckOutDate = HotelSearch.CheckOutDate,
        Hotel = SelectedHotel,
        ApplicationUser = CurrentUser
    };

    _dbContext.HotelBookings.Add(hotelBooking);
    await _dbContext.SaveChangesAsync();

    return RedirectToPage("/Payment", new
    {
        bookingId = hotelBooking.HotelBookingId.ToString(),
        bookingType = "hotel"
    });
}
```

For more detail, see Appendix [5.3](#53-files-for-hotel-tour-and-package-bookings) for the content of all files relating to bookings.

## 3.3 Key requirement 3: View Bookings

For this requirement, any hotels, tours or packages booked by the user should be displayed promptly after a successful payment. This page should hold the latest details of bookings, so even if they are edited (see [3.4](#34-key-requirement-4-edit-bookings)) the most up-to-date details should be displayed.

Starting with the client side code, again, just looking at the hotels should suffice since tours and packages follow the same logic.

Here, inspired by the Bookings page, the `asp-page-handler` element ensures that on form submit, the URL handler parameter is `HotelTable`, helping it to be differentiated from the other forms on the page. The UI is tabular based, so it takes on the headings similar to the properties of the `Hotel` model class. Using in line C#, the rows can be dynamically updated via a `foreach` loop that iterated through a list of type `HotelBookings` that is populated dynamically in the server-side. The row for the hotel cost does a calculation, multiplying it by the duration of days. The days obviously being the difference between the `HotelBooking.CheckInDate` and the `HotelBooking.CheckOutDate`.

```html
<form id="hotelTableForm" method="post" asp-page-handler="HotelTable">
  <h3>Hotels</h3>
  <div class="container">
    <div class="row">
      <div class="col-md-12">
        <table class="table">
          <thead>
            <tr>
              <th>Name</th>
              <th>Room Type</th>
              <th>Check-in Date</th>
              <th>Check-out Date</th>
              <th>Cost</th>
            </tr>
          </thead>
          <tbody>
            <input
              type="hidden"
              id="hotelBookingIdInput"
              name="hotelBookingId"
              value=""
            />
            @foreach (var item in Model.ViewBookingsTable.HotelBookingsList) {
            <tr>
              <td>@item.Hotel.Name</td>
              <td>@item.Hotel.RoomType</td>
              <td>@item.CheckInDate.ToShortDateString()</td>
              <td>@item.CheckOutDate.ToShortDateString()</td>
              <td>
                @("£" + ((item.CheckOutDate - item.CheckInDate).Days *
                item.Hotel.Cost).ToString("0.00"))
              </td>
              <td>
                <div class="btn-group" role="group">
                  <input
                    type="submit"
                    class="btn btn-primary"
                    name="command"
                    value="Edit"
                    onclick="submitForm('hotelBookingIdInput', 'hotelTableForm', '@item.HotelBookingId')"
                  />
                </div>
                <div class="btn-group" role="group">
                  <input
                    type="submit"
                    class="btn btn-danger"
                    name="command"
                    value="Cancel"
                    onclick="onCancelClick('hotelBookingIdInput', 'hotelTableForm', '@item.HotelBookingId')"
                  />
                </div>
              </td>
            </tr>
            }
          </tbody>
        </table>
      </div>
    </div>
  </div>
</form>
```

For each table entry, there are two additional buttons. One for editing and the other for the cancellation of bookings. These buttons are bound to the below JavaScript functions. Upon the cancellation button, a dialog box appears asking the user for confirmation. It then dynamically submits the form based on that, via the `submitForm` function.

```javascript
function onCancelClick(bookingInputId, submitFormId, bookingId) {
  var isConfirmed = window.confirm(
    "Are you sure you want to cancel your booking?"
  );

  if (isConfirmed) {
    submitForm(bookingInputId, submitFormId, bookingId);
  }
}

function submitForm(bookingInputId, submitFormId, bookingId) {
  var form = document.getElementById(submitFormId);
  document.getElementById(bookingInputId).value = bookingId;
  form.submit();
}
```

For the server-side component for this page, has an `OnGet` function, that executes code on page reload. First, the ID of the current user is required, and using this, a query to the database is made to retrieve all relevant bookings, of all types i.e. hotels, tours and packages. The variable which are bound to the client side take on the value of these variables so that it can be displayed on the page. Finally, the `successMessage` element takes on the query parameter in the URL from the payments page to provide user feedback.

```csharp
public async Task<IActionResult> OnGet()
{
    var CurrentUser = await _userManager.GetUserAsync(User);

    var hotelBookingsList = await _dbContext.HotelBookings
        .Where(hb => hb.UserId.Equals(CurrentUser.Id) && hb.IsCancelled == false)
        .Include(hb => hb.Hotel)
        .ToListAsync();

    ViewBookingsTable.HotelBookingsList = hotelBookingsList;

    var tourBookingsList = await _dbContext.TourBookings
        .Where(tb => tb.UserId.Equals(CurrentUser.Id) && tb.IsCancelled == false)
        .Include(tb => tb.Tour)
        .ToListAsync();

    ViewBookingsTable.TourBookingsList = tourBookingsList;

    var packageBookingsList = await _dbContext.PackageBookings
        .Where(pb => pb.UserId.Equals(CurrentUser.Id) && pb.IsCancelled == false)
        .Include(pb => pb.Hotel)
        .Include(pb => pb.Tour)
        .ToListAsync();

    ViewBookingsTable.PackageBookingsList = packageBookingsList;

    ViewBookingsTable.SuccessMessage = Request.Query["successMessage"];

    return Page();
}
```

Each table in the ViewBookings page has its own `PostAsync` function with the same logic as demonstrated below. It first attempts to take the `HotelBookingId` from the client-side form to make a request to the database to retrieve the correct `HotelBooking`. The `IsCancelled` property is set to the boolean value of `true` and the page redirects to "/ViewBookings". However if the "Edit" button is clicked, it redirects to the appropriate EditBooking page (see [3.4](#34-key-requirement-4-edit-bookings)) and passes the appropriate `HotelBookingId` as a URL query parameter.

```csharp
public async Task<IActionResult> OnPostHotelTableAsync(string command, string returnUrl = null)
{
    if (command == "Cancel")
    {
        var HotelBookingId = new Guid(Request.Form["hotelBookingId"]);

        var hotelBooking = await _dbContext.HotelBookings
            .Where(hb => hb.HotelBookingId == HotelBookingId)
            .FirstOrDefaultAsync();

        hotelBooking.IsCancelled = true;

        await _dbContext.SaveChangesAsync();

        return RedirectToPage("/ViewBookings");
    }
    else
    {
        return RedirectToPage("/EditHotelBooking", new
        {
            hotelBookingId = Request.Form["hotelBookingId"]
        });
    }
}
```

For more detail, see Appendix [5.4](#54-files-for-view-bookings) for the content of all files relating to view bookings.

## 3.4 Key requirement 4: Edit Bookings

For more detail, see Appendix () for the content of all files relating to edit bookings.

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

## 5.3 Files for hotel, tour, and package Bookings

### 5.3.1 W3Schools tab-based UI

```html
<!DOCTYPE html>
<html>
  <head>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style>
      body {
        font-family: Arial;
      }

      /* Style the tab */
      .tab {
        overflow: hidden;
        border: 1px solid #ccc;
        background-color: #f1f1f1;
      }

      /* Style the buttons inside the tab */
      .tab button {
        background-color: inherit;
        float: left;
        border: none;
        outline: none;
        cursor: pointer;
        padding: 14px 16px;
        transition: 0.3s;
        font-size: 17px;
      }

      /* Change background color of buttons on hover */
      .tab button:hover {
        background-color: #ddd;
      }

      /* Create an active/current tablink class */
      .tab button.active {
        background-color: #ccc;
      }

      /* Style the tab content */
      .tabcontent {
        display: none;
        padding: 6px 12px;
        border: 1px solid #ccc;
        border-top: none;
      }
    </style>
  </head>
  <body>
    <h2>Tabs</h2>
    <p>Click on the buttons inside the tabbed menu:</p>

    <div class="tab">
      <button class="tablinks" onclick="openCity(event, 'London')">
        London
      </button>
      <button class="tablinks" onclick="openCity(event, 'Paris')">Paris</button>
      <button class="tablinks" onclick="openCity(event, 'Tokyo')">Tokyo</button>
    </div>

    <div id="London" class="tabcontent">
      <h3>London</h3>
      <p>London is the capital city of England.</p>
    </div>

    <div id="Paris" class="tabcontent">
      <h3>Paris</h3>
      <p>Paris is the capital of France.</p>
    </div>

    <div id="Tokyo" class="tabcontent">
      <h3>Tokyo</h3>
      <p>Tokyo is the capital of Japan.</p>
    </div>

    <script>
      function openCity(evt, cityName) {
        var i, tabcontent, tablinks;
        tabcontent = document.getElementsByClassName("tabcontent");
        for (i = 0; i < tabcontent.length; i++) {
          tabcontent[i].style.display = "none";
        }
        tablinks = document.getElementsByClassName("tablinks");
        for (i = 0; i < tablinks.length; i++) {
          tablinks[i].className = tablinks[i].className.replace(" active", "");
        }
        document.getElementById(cityName).style.display = "block";
        evt.currentTarget.className += " active";
      }
    </script>
  </body>
</html>
```

### 5.3.2 [`Bookings.cshtml`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Pages/Bookings.cshtml)

```csharp
@page
@model BookingsModel
@{
    ViewData["Title"] = "Bookings";
}

<h2>Bookings</h2>
<style>
    .tab {
        overflow: hidden;
        border: 1px solid #ccc;
        background-color: #f1f1f1;
    }

        .tab button {
            background-color: inherit;
            float: left;
            border: none;
            outline: none;
            cursor: pointer;
            padding: 14px 16px;
            transition: 0.3s;
            font-size: 17px;
        }

            .tab button:hover {
                background-color: #ddd;
            }

            .tab button.active {
                background-color: #ccc;
            }

    .tabcontent {
        display: none;
        padding: 6px 12px;
        border: 1px solid #ccc;
        border-top: none;
    }
</style>
<div class="tab">
    <button class="tablinks" onclick="openTab(event, 'Hotels')" id="HotelTab">Hotels</button>
    <button class="tablinks" onclick="openTab(event, 'Tours')" id="TourTab">Tours</button>
    <button class="tablinks" onclick="openTab(event, 'Packages')" id="PackageTab">Packages</button>
</div>
<div asp-validation-summary="ModelOnly" class="text-danger" role="alert"></div>
<form id="hotelSearchForm" method="post" asp-page-handler="HotelSearch">
    <div id="Hotels" class="tabcontent">
        <h3>Search Hotels</h3>
        <hr />
        <div class="form-group">
            <label asp-for="HotelSearch.CheckInDate">Select check-in date:</label>
            <input id="hotelCheckInDate" asp-for="HotelSearch.CheckInDate" type="date" class="form-control" value="2024-01-09" />
            <span asp-validation-for="HotelSearch.CheckInDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label asp-for="HotelSearch.CheckInDate">Select check-out date:</label>
            <input id="hotelCheckOutDate" asp-for="HotelSearch.CheckOutDate" type="date" class="form-control" value="2024-01-27" />
            <span asp-validation-for="HotelSearch.CheckOutDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label asp-for="HotelSearch.RoomType">Select room type:</label>
            <select asp-for="HotelSearch.RoomType" asp-items="@Model.HotelSearch.RoomTypes" class="form-control"></select>
            <span asp-validation-for="HotelSearch.RoomType" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label for="hotels">Hotels:</label>
            <select name="hotels" id="hotelsDropdown" onchange='calculateTotalCost("hotelsDropdown", "hotelCheckInDate", "hotelCheckOutDate", "hotelTotalCostMessage")'>
                @foreach (var hotel in Model.HotelSearch.HotelsList)
                {
                    <option value="@hotel.HotelId">@(hotel.Name + " £" + hotel.Cost + " (per night)")</option>
                }
            </select>
            <p id="hotelsDropdownErrorMessage" class="text-danger"></p>
        </div>
        <div class="form-group">
            <input type="submit" name="command" class="btn btn-primary" value="Search" />
        </div>
        <hr />
        <p id="hotelTotalCostMessage">Total Cost: £0</p>
        <div class="form-group">
            <input class="btn btn-primary" name="command" value="Book" onclick="submitHotelSearchForm()" />
        </div>
    </div>
</form>
<div asp-validation-summary="ModelOnly" class="text-danger" role="alert"></div>
<form id="tourSearchForm" method="post" asp-page-handler="TourSearch">
    <div id="Tours" class="tabcontent">
        <h3>Search Tours</h3>
        <hr />
        <div class="form-group">
            <label asp-for="TourSearch.TourStartDate">Select Tour Start Date:</label>
            <input id="tourStartDate" asp-for="TourSearch.TourStartDate" type="date" class="form-control" value="2024-01-15" />
            <span asp-validation-for="TourSearch.TourEndDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label asp-for="TourSearch.TourEndDate">Select Tour End Date:</label>
            <input id="tourEndDate"  asp-for="TourSearch.TourEndDate" type="date" class="form-control" value="2024-01-20" />
            <span asp-validation-for="TourSearch.TourEndDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label for="tours">Tours:</label>
            <select name="tours" id="toursDropdown" onchange='calculateTotalCost("toursDropdown", "tourStartDate", "tourEndDate", "tourTotalCostMessage")'>
                @foreach (var tour in Model.TourSearch.ToursList)
                {
                    <option value="@tour.TourId">@(tour.Name + " £" + tour.Cost)</option>
                }
            </select>
            <p id="toursDropdownErrorMessage" class="text-danger"></p>
        </div>
        <div class="form-group">
            <input type="submit" name="command" class="btn btn-primary" value="Search"/>
        </div>
        <hr />
        <p id="tourTotalCostMessage">Total Cost: £0</p>
        <div class="form-group">
            <input class="btn btn-primary" name="command" value="Book" onclick="submitTourSearchForm()"/>
        </div>
    </div>
</form>
<div asp-validation-summary="ModelOnly" class="text-danger" role="alert"></div>
<form id="packageBookForm" method="post" asp-page-handler="PackageBook">
    <div id="Packages" class="tabcontent">
        <h3>Packages</h3>
        <hr />
        <h4>Search Hotels</h4>
        <hr />
        <div class="form-group">
            <label asp-for="PackageBook.CheckInDate">Select check-in date:</label>
            <input id="packageHotelCheckInDate" asp-for="PackageBook.CheckInDate" type="date" class="form-control" value="2024-01-09" />
            <span asp-validation-for="PackageBook.CheckInDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label asp-for="PackageBook.CheckInDate">Select check-out date:</label>
            <input id="packageHotelCheckOutDate" asp-for="PackageBook.CheckOutDate" type="date" class="form-control" value="2024-01-27" />
            <span asp-validation-for="PackageBook.CheckOutDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label asp-for="PackageBook.RoomType">Select room type:</label>
            <select asp-for="PackageBook.RoomType" asp-items="@Model.PackageBook.RoomTypes" class="form-control"></select>
            <span asp-validation-for="PackageBook.RoomType" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label for="packageHotels">Hotels:</label>
            <select name="packageHotelsDropdown" id="packageHotelsDropdownId" onchange='calculateTotalPackageCost()'>
                @foreach (var hotel in Model.PackageBook.HotelsList)
                {
                    <option value="@hotel.HotelId">@(hotel.Name + " £" + hotel.Cost)</option>
                }
            </select>
            <p id="packageHotelsDropdownErrorMessage" class="text-danger"></p>
        </div>
        <hr />
        <h4>Search Tours</h4>
        <hr />
        <div class="form-group">
            <label asp-for="PackageBook.TourStartDate">Select Tour Start Date:</label>
            <input id="packageTourStartDate" asp-for="PackageBook.TourStartDate" type="date" class="form-control" value="2024-01-15" />
            <span asp-validation-for="PackageBook.TourEndDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label asp-for="PackageBook.TourEndDate">Select Tour End Date:</label>
            <input id="packageTourEndDate" asp-for="PackageBook.TourEndDate" type="date" class="form-control" value="2024-01-20" />
            <span asp-validation-for="PackageBook.TourEndDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label for="packageTours">Tours:</label>
            <select name="packageToursDropdown" id="packageToursDropdownId" onchange='calculateTotalPackageCost()'>
                @foreach (var tour in Model.PackageBook.ToursList)
                {
                    <option value="@tour.TourId">@(tour.Name + " £" + tour.Cost)</option>
                }
            </select>
            <p id="packageToursDropdownErrorMessage" class="text-danger"></p>
        </div>
        <hr />
        <div class="form-group">
            <input type="submit" name="command" class="btn btn-primary" value="Search" />
        </div>
        <hr />
        <p id="packageTotalCostMessage">Total Cost: £0</p>
        <div class="form-group">
            <input class="btn btn-primary" name="command" value="Book" onclick="submitPackageBookForm()" />
        </div>
    </div>
</form>
<script>
    function openTab(evt, tabName) {
        var i, tabcontent, tablinks;
        tabcontent = document.getElementsByClassName("tabcontent");
        for (i = 0; i < tabcontent.length; i++) {
            tabcontent[i].style.display = "none";
        }
        tablinks = document.getElementsByClassName("tablinks");
        for (i = 0; i < tablinks.length; i++) {
            tablinks[i].className = tablinks[i].className.replace(" active", "");
        }
        document.getElementById(tabName).style.display = "block";
        evt.currentTarget.className += " active";
    }

    let params = (new URL(document.location)).searchParams;
    let handler = params.get("handler");

    switch (handler) {
        case "HotelSearch":
            document.getElementById("HotelTab").click();
            break;
        case "TourSearch":
            document.getElementById("TourTab").click();
            break;
        case "PackageBook":
            document.getElementById("PackageTab").click();
            break;
        default:
            break;
    }

    function submitHotelSearchForm() {
        var dropdownErrorMessage = document.getElementById("hotelsDropdownErrorMessage");
        dropdownErrorMessage.innerHTML = "";

        var form = document.getElementById("hotelSearchForm");
        var selectBoxValue = form.elements["hotelsDropdown"].value;

        if (!selectBoxValue) {
            dropdownErrorMessage.innerHTML = "Please select a hotel for booking";
            return;
        }

        form.elements["command"].value = "Book";
        form.submit();
    }

    function submitTourSearchForm() {
        var dropdownErrorMessage = document.getElementById("toursDropdownErrorMessage");
        dropdownErrorMessage.innerHTML = "";

        var form = document.getElementById("tourSearchForm");
        var selectBoxValue = form.elements["toursDropdown"].value;

        if (!selectBoxValue) {
            dropdownErrorMessage.innerHTML = "Please select a tour for booking";
            return;
        }

        form.elements["command"].value = "Book";
        form.submit();
    }

    function submitPackageBookForm() {
        var hotelsDropdownErrorMessage = document.getElementById("packageHotelsDropdownErrorMessage");
        hotelsDropdownErrorMessage.innerHTML = "";

        var toursDropdownErrorMessage = document.getElementById("packageToursDropdownErrorMessage");
        toursDropdownErrorMessage.innerHTML = "";

        var form = document.getElementById("packageBookForm");

        var hotelsSelectBoxValue = form.elements["packageHotelsDropdown"].value;
        var toursSelectBoxValue = form.elements["packageToursDropdown"].value;

        var errorReturn = false;

        if (!hotelsSelectBoxValue) {
            hotelsDropdownErrorMessage.innerHTML = "Please select a hotel for booking";
            errorReturn = true;
        }

        if (!toursSelectBoxValue) {
            toursDropdownErrorMessage.innerHTML = "Please select a tour for booking";
            errorReturn = true;
        }

        if (errorReturn) {
            return;
        }

        form.elements["command"].value = "Book";
        form.submit();
    }

    function calculateTotal(dropdown, fromDate, toDate) {
        var selectedDropdown = document.getElementById(dropdown);
        var selectedOption = selectedDropdown.options[selectedDropdown.selectedIndex];
        var cost = selectedOption.textContent.split(' £')[1].split(' ')[0];

        var startDate = new Date(document.getElementById(fromDate).value);
        var endDate = new Date(document.getElementById(toDate).value);

        var durationInDays = Math.ceil((endDate - startDate) / (1000 * 60 * 60 * 24));

        return parseFloat((cost * durationInDays).toFixed(2));
    }

    function calculateTotalCost(dropdown, fromDate, toDate, totalCostMessageId) {
        document.getElementById(totalCostMessageId).innerHTML = "Total Cost: £" + calculateTotal(dropdown, fromDate, toDate);
    }

    function calculateTotalPackageCost() {
        var packageHotelCost = calculateTotal("packageHotelsDropdownId", "packageHotelCheckInDate", "packageHotelCheckOutDate");
        var packageTourCost = calculateTotal("packageToursDropdownId", "packageTourStartDate", "packageTourEndDate");

        var packageTotalCost = packageHotelCost + packageTourCost

        document.getElementById("packageTotalCostMessage").innerHTML = "Total Cost: £" + packageTotalCost;
    }

    window.addEventListener('load', function () {
        if (document.getElementById("HotelTab").className == "tablinks active") {
            calculateTotalCost("hotelsDropdown", "hotelCheckInDate", "hotelCheckOutDate", "hotelTotalCostMessage");
        }
        else if (document.getElementById("TourTab").className == "tablinks active") {
            calculateTotalCost("toursDropdown", "tourStartDate", "tourEndDate", "tourTotalCostMessage");
        }
        else if (document.getElementById("PackageTab").className == "tablinks active") {
            calculateTotalPackageCost();
        }
    });
</script>
```

### 5.3.3 [`Bookings.cshtml.cs`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Pages/Bookings.cshtml.cs)

```csharp
using asp_net_core_web_app_authentication_authorisation.Models;
using asp_net_core_web_app_authentication_authorisation.Services;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.EntityFrameworkCore;
using System.ComponentModel.DataAnnotations;

namespace asp_net_core_web_app_authentication_authorisation.Pages
{
    public class BookingsModel : PageModel
    {
        [BindProperty]
        public HotelSearchModel HotelSearch { get; set; }

        [BindProperty]
        public TourSearchModel TourSearch { get; set; }

        [BindProperty]
        public PackageBookModel PackageBook { get; set; }

        private readonly ApplicationDbContext _dbContext;
        private readonly UserManager<ApplicationUser> _userManager;

        public BookingsModel(ApplicationDbContext dbContext, UserManager<ApplicationUser> userManager)
        {
            HotelSearch = new HotelSearchModel();
            TourSearch = new TourSearchModel();
            PackageBook = new PackageBookModel();
            _dbContext = dbContext;
            _userManager = userManager;
        }

        public class HotelSearchModel
        {
            [Required(ErrorMessage = "Please select a check-in date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Check in date")]
            public DateTime CheckInDate { get; set; }

            [Required(ErrorMessage = "Please select a check-out date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Check out date")]
            public DateTime CheckOutDate { get; set; }

            [Required(ErrorMessage = "Please select a room type")]
            [DataType(DataType.Text)]
            [Display(Name = "Room type")]
            public string RoomType { get; set; } = "Single";

            public List<SelectListItem> RoomTypes { get; set; } = new List<SelectListItem>
            {
                new SelectListItem
                {
                    Value = "single",
                    Text = "Single"
                },
                new SelectListItem
                {
                    Value = "double",
                    Text = "Double"
                },
                new SelectListItem
                {
                    Value = "family suite",
                    Text = "Family Suite"
                }
            };

            public List<Hotel> HotelsList { get; set; } = new List<Hotel>();
        }

        public class TourSearchModel
        {
            [Required(ErrorMessage = "Please select a tour start date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Tour start date")]
            public DateTime TourStartDate { get; set; }

            [Required(ErrorMessage = "Please select a tour end date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Tour end date")]
            public DateTime TourEndDate { get; set; }

            public List<String> AvailableTours { get; set; } = new List<string>();

            public List<Tour> ToursList { get; set; } = new List<Tour>();
        }

        public class PackageBookModel
        {
            [Required(ErrorMessage = "Please select a check-in date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Check in date")]
            public DateTime CheckInDate { get; set; }

            [Required(ErrorMessage = "Please select a check-out date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Check out date")]
            public DateTime CheckOutDate { get; set; }

            [Required(ErrorMessage = "Please select a room type")]
            [DataType(DataType.Text)]
            [Display(Name = "Room type")]
            public string RoomType { get; set; } = "Single";

            public List<SelectListItem> RoomTypes { get; set; } = new List<SelectListItem>
            {
                new SelectListItem
                {
                    Value = "single",
                    Text = "Single"
                },
                new SelectListItem
                {
                    Value = "double",
                    Text = "Double"
                },
                new SelectListItem
                {
                    Value = "family suite",
                    Text = "Family Suite"
                }
            };

            public List<Hotel> HotelsList { get; set; } = new List<Hotel>();

            [Required(ErrorMessage = "Please select a tour start date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Tour start date")]
            public DateTime TourStartDate { get; set; }

            [Required(ErrorMessage = "Please select a tour end date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Tour end date")]
            public DateTime TourEndDate { get; set; }

            public List<Tour> ToursList { get; set; } = new List<Tour>();
        }

        public async Task<IActionResult> OnPostHotelSearchAsync(string command, string returnUrl = null)
        {
            if (!ModelState.IsValid)
            {
                return Page();
            }

            if (command == "Search")
            {
                var availableHotels = await _dbContext.HotelAvailabilities
                    .Where(ha =>
                        ha.AvailableFrom <= HotelSearch.CheckInDate &&
                        ha.AvailableTo >= HotelSearch.CheckOutDate &&
                        ha.Hotel.RoomType == HotelSearch.RoomType)
                    .Select(ha => ha.Hotel)
                    .Distinct()
                    .ToListAsync();

                HotelSearch.HotelsList = availableHotels;

                return Page();
            }
            else
            {
                var SelectedHotelId = new Guid(Request.Form["hotels"]);
                var CurrentUser = await _userManager.GetUserAsync(User);
                Hotel SelectedHotel = await _dbContext.Hotels.FindAsync(SelectedHotelId);

                var hotelBooking = new HotelBooking
                {
                    HotelBookingId = new Guid(),
                    HotelId = SelectedHotelId,
                    UserId = CurrentUser.Id,
                    CheckInDate = HotelSearch.CheckInDate,
                    CheckOutDate = HotelSearch.CheckOutDate,
                    Hotel = SelectedHotel,
                    ApplicationUser = CurrentUser
                };

                _dbContext.HotelBookings.Add(hotelBooking);
                await _dbContext.SaveChangesAsync();

                return RedirectToPage("/Payment", new
                {
                    bookingId = hotelBooking.HotelBookingId.ToString(),
                    bookingType = "hotel"
                });
            }
        }

        public async Task<IActionResult> OnPostTourSearchAsync(string command, string returnUrl = null)
        {
            if (!ModelState.IsValid)
            {
                return Page();
            }

            if (command == "Search")
            {
                var availableTours = await _dbContext.TourAvailabilities
                .Where(ta =>
                    ta.AvailableFrom <= TourSearch.TourStartDate && ta.AvailableTo >= TourSearch.TourEndDate)
                .Select(ta => ta.Tour)
                .Distinct()
                .ToListAsync();

                TourSearch.ToursList = availableTours;

                return Page();
            }
            else
            {
                var SelectedTourId = new Guid(Request.Form["tours"]);
                var CurrentUser = await _userManager.GetUserAsync(User);
                Tour SelectedTour = await _dbContext.Tours.FindAsync(SelectedTourId);

                var tourBooking = new TourBooking
                {
                    TourBookingId = new Guid(),
                    TourId = SelectedTourId,
                    UserId = CurrentUser.Id,
                    TourStartDate = TourSearch.TourStartDate,
                    TourEndDate = TourSearch.TourEndDate,
                    Tour = SelectedTour,
                    ApplicationUser = CurrentUser
                };

                _dbContext.TourBookings.Add(tourBooking);
                await _dbContext.SaveChangesAsync();

                return RedirectToPage("/Payment", new
                {
                    bookingId = tourBooking.TourBookingId.ToString(),
                    bookingType = "tour"
                });
            }
        }

        public async Task<IActionResult> OnPostPackageBookAsync(string command, string returnUrl = null)
        {
            if (!ModelState.IsValid)
            {
                return Page();
            }

            if (command == "Search")
            {
                var availableHotels = await _dbContext.HotelAvailabilities
                .Where(ha =>
                    ha.AvailableFrom <= PackageBook.CheckInDate && ha.AvailableTo >= PackageBook.CheckOutDate)
                .Select(ha => ha.Hotel)
                .Distinct()
                .ToListAsync();

                PackageBook.HotelsList = availableHotels;

                var availableTours = await _dbContext.TourAvailabilities
                    .Where(ta =>
                        ta.AvailableFrom <= PackageBook.TourStartDate && ta.AvailableTo >= PackageBook.TourEndDate)
                    .Select(ta => ta.Tour)
                    .Distinct()
                    .ToListAsync();

                PackageBook.ToursList = availableTours;

                return Page();
            }
            else
            {
                var CurrentUser = await _userManager.GetUserAsync(User);

                var SelectedHotelId = new Guid(Request.Form["packageHotelsDropdown"]);
                Hotel SelectedHotel = await _dbContext.Hotels.FindAsync(SelectedHotelId);

                var SelectedTourId = new Guid(Request.Form["packageToursDropdown"]);
                Tour SelectedTour = await _dbContext.Tours.FindAsync(SelectedTourId);

                var packageBooking = new PackageBooking
                {
                    PackageBookingId = new Guid(),
                    UserId = CurrentUser.Id,
                    HotelId = SelectedHotelId,
                    CheckInDate = PackageBook.CheckInDate,
                    CheckOutDate = PackageBook.CheckOutDate,
                    TourId = SelectedTourId,
                    TourStartDate = PackageBook.TourStartDate,
                    TourEndDate = PackageBook.TourEndDate,
                    Hotel = SelectedHotel,
                    Tour = SelectedTour,
                    ApplicationUser = CurrentUser
                };

                _dbContext.PackageBookings.Add(packageBooking);

                await _dbContext.SaveChangesAsync();

                return RedirectToPage("/Payment", new
                {
                    bookingId = packageBooking.PackageBookingId.ToString(),
                    bookingType = "package"
                });
            }
        }
    }
}
```

## 5.4 Files for View Bookings

### 5.4.1 [`ViewBookings.cshtml`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Pages/ViewBookings.cshtml)

```csharp
@page
@model ViewBookingsModel
@{
    ViewData["Title"] = "ViewBookings";
}

<h2>Your bookings</h2>
<p>Any bookings you have made will be displayed here</p>
<hr />
<p class="text-success">@Model.ViewBookingsTable.SuccessMessage</p>
<form id="hotelTableForm" method="post" asp-page-handler="HotelTable">
    <h3>Hotels</h3>
    <div class="container">
        <div class="row">
            <div class="col-md-12">
                <table class="table">
                    <thead>
                        <tr>
                            <th>Name</th>
                            <th>Room Type</th>
                            <th>Check-in Date</th>
                            <th>Check-out Date</th>
                            <th>Cost</th>
                        </tr>
                    </thead>
                    <tbody>
                        <input type="hidden" id="hotelBookingIdInput" name="hotelBookingId" value="">
                        @foreach (var item in Model.ViewBookingsTable.HotelBookingsList)
                        {
                            <tr>
                                <td>@item.Hotel.Name</td>
                                <td>@item.Hotel.RoomType</td>
                                <td>@item.CheckInDate.ToShortDateString()</td>
                                <td>@item.CheckOutDate.ToShortDateString()</td>
                                <td>@("£" + ((item.CheckOutDate - item.CheckInDate).Days * item.Hotel.Cost).ToString("0.00"))</td>
                                <td>
                                    <div class="btn-group" role="group">
                                        <input type="submit" class="btn btn-primary" name="command" value="Edit" onclick="submitForm('hotelBookingIdInput', 'hotelTableForm', '@item.HotelBookingId')" />
                                    </div>
                                    <div class="btn-group" role="group">
                                        <input type="submit" class="btn btn-danger" name="command" value="Cancel" onclick="onCancelClick('hotelBookingIdInput', 'hotelTableForm', '@item.HotelBookingId')" />
                                    </div>
                                </td>
                            </tr>
                        }
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</form>
<hr />
<form id="tourTableForm" method="post" asp-page-handler="TourTable">
    <h3>Tours</h3>
    <div class="container">
        <div class="row">
            <div class="col-md-12">
                <table class="table">
                    <thead>
                        <tr>
                            <th>Name</th>
                            <th>Duration (Days)</th>
                            <th>Start Date</th>
                            <th>End Date</th>
                            <th>Cost</th>
                        </tr>
                    </thead>
                    <tbody>
                        <input type="hidden" id="tourBookingIdInput" name="tourBookingId" value="">
                        @foreach (var item in Model.ViewBookingsTable.TourBookingsList)
                        {
                            <tr>
                                <td>@item.Tour.Name</td>
                                <td>@item.Tour.DurationInDays</td>
                                <td>@item.TourStartDate.ToShortDateString()</td>
                                <td>@item.TourEndDate.ToShortDateString()</td>
                                <td>@("£" + item.Tour.Cost)</td>
                                <td>
                                    <div class="btn-group" role="group">
                                        <input type="submit" class="btn btn-primary" name="command" value="Edit" onclick="submitForm('tourBookingIdInput', 'tourTableForm', '@item.TourBookingId')" />
                                    </div>
                                    <div class="btn-group" role="group">
                                        <input type="submit" class="btn btn-danger" name="command" value="Cancel" onclick="onCancelClick('tourBookingIdInput', 'tourTableForm', '@item.TourBookingId')" />
                                    </div>
                                </td>
                            </tr>
                        }
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</form>
<hr />
<form id="packageTableForm" method="post" asp-page-handler="PackageTable">
    <h3>Packages</h3>
    <div class="container">
        <div class="row">
            <div class="col-md-12">
                <table class="table">
                    <thead>
                        <tr>
                            <th>Hotel</th>
                            <th>Room Type</th>
                            <th>Check-in Date</th>
                            <th>Check-out Date</th>
                            <th>Tour</th>
                            <th>Duration (Days)</th>
                            <th>Start Date</th>
                            <th>End Date</th>
                            <th>Total Cost</th>
                        </tr>
                    </thead>
                    <tbody>
                        <input type="hidden" id="packageBookingIdInput" name="packageBookingId" value="">
                        @foreach (var item in Model.ViewBookingsTable.PackageBookingsList)
                        {
                            <tr>
                                <td>@item.Hotel.Name</td>
                                <td>@item.Hotel.RoomType</td>
                                <td>@item.CheckInDate.ToShortDateString()</td>
                                <td>@item.CheckOutDate.ToShortDateString()</td>
                                <td>@item.Tour.Name</td>
                                <td>@item.Tour.DurationInDays</td>
                                <td>@item.TourStartDate.ToShortDateString()</td>
                                <td>@item.TourEndDate.ToShortDateString()</td>
                                <td>
                                    @{
                                        var hotelCost = (item.CheckOutDate - item.CheckInDate).Days * item.Hotel.Cost;
                                        var tourCost = (item.TourEndDate - item.TourStartDate).Days * item.Tour.Cost;
                                        var totalCost = hotelCost + tourCost;
                                    }
                                    @("£" + totalCost.ToString("0.00"))
                                </td>
                                <td>
                                    <div class="btn-group" role="group">
                                        <input type="submit" class="btn btn-primary" name="command" value="Edit" onclick="submitForm('packageBookingIdInput', 'packageTableForm', '@item.PackageBookingId')" />
                                    </div>
                                    <div class="btn-group" role="group">
                                        <input type="submit" class="btn btn-danger" name="command" value="Cancel" onclick="onCancelClick('packageBookingIdInput', 'packageTableForm', '@item.PackageBookingId')" />
                                    </div>
                                </td>
                            </tr>
                        }
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</form>
<script>
    function onCancelClick(bookingInputId, submitFormId, bookingId) {
        var isConfirmed = window.confirm("Are you sure you want to cancel your booking?");

        if (isConfirmed) {
            submitForm(bookingInputId, submitFormId, bookingId)
        }
    }

    function submitForm(bookingInputId, submitFormId, bookingId) {
        var form = document.getElementById(submitFormId);
        document.getElementById(bookingInputId).value = bookingId;
        form.submit();
    }
</script>
```

### 5.4.2 [`ViewBookings.cshtml.cs`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Pages/ViewBookings.cshtml.cs)

```csharp
using asp_net_core_web_app_authentication_authorisation.Models;
using asp_net_core_web_app_authentication_authorisation.Services;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.EntityFrameworkCore;

namespace asp_net_core_web_app_authentication_authorisation.Pages
{
    public class ViewBookingsModel : PageModel
    {
        [BindProperty]
        public ViewBookingsTableModel ViewBookingsTable { get; set; }

        private readonly ApplicationDbContext _dbContext;
        private readonly UserManager<ApplicationUser> _userManager;

        public ViewBookingsModel(ApplicationDbContext dbContext, UserManager<ApplicationUser> userManager)
        {
            ViewBookingsTable = new ViewBookingsTableModel();
            _dbContext = dbContext;
            _userManager = userManager;
        }

        public class ViewBookingsTableModel
        {
            public List<HotelBooking> HotelBookingsList { get; set; } = new List<HotelBooking>();
            public List<TourBooking> TourBookingsList { get; set; } = new List<TourBooking>();
            public List<PackageBooking> PackageBookingsList { get; set; } = new List<PackageBooking>();

            public string SuccessMessage { get; set; }
        }

        public async Task<IActionResult> OnGet()
        {
            var CurrentUser = await _userManager.GetUserAsync(User);

            var hotelBookingsList = await _dbContext.HotelBookings
                .Where(hb => hb.UserId.Equals(CurrentUser.Id) && hb.IsCancelled == false)
                .Include(hb => hb.Hotel)
                .ToListAsync();

            ViewBookingsTable.HotelBookingsList = hotelBookingsList;

            var tourBookingsList = await _dbContext.TourBookings
                .Where(tb => tb.UserId.Equals(CurrentUser.Id) && tb.IsCancelled == false)
                .Include(tb => tb.Tour)
                .ToListAsync();

            ViewBookingsTable.TourBookingsList = tourBookingsList;

            var packageBookingsList = await _dbContext.PackageBookings
                .Where(pb => pb.UserId.Equals(CurrentUser.Id) && pb.IsCancelled == false)
                .Include(pb => pb.Hotel)
                .Include(pb => pb.Tour)
                .ToListAsync();

            ViewBookingsTable.PackageBookingsList = packageBookingsList;

            ViewBookingsTable.SuccessMessage = Request.Query["successMessage"];

            return Page();
        }

        public async Task<IActionResult> OnPostHotelTableAsync(string command, string returnUrl = null)
        {
            if (command == "Cancel")
            {
                var HotelBookingId = new Guid(Request.Form["hotelBookingId"]);

                var hotelBooking = await _dbContext.HotelBookings
                    .Where(hb => hb.HotelBookingId == HotelBookingId)
                    .FirstOrDefaultAsync();

                hotelBooking.IsCancelled = true;

                await _dbContext.SaveChangesAsync();

                return RedirectToPage("/ViewBookings");
            }
            else
            {
                return RedirectToPage("/EditHotelBooking", new
                {
                    hotelBookingId = Request.Form["hotelBookingId"]
                });
            }
        }

        public async Task<IActionResult> OnPostTourTableAsync(string command, string returnUrl = null)
        {
            if (command == "Cancel")
            {
                var TourBookingId = new Guid(Request.Form["tourBookingId"]);

                var tourBooking = await _dbContext.TourBookings
                    .Where(tb => tb.TourBookingId == TourBookingId)
                    .FirstOrDefaultAsync();

                tourBooking.IsCancelled = true;

                await _dbContext.SaveChangesAsync();

                return RedirectToPage("/ViewBookings");
            }
            else
            {
                return RedirectToPage("/EditTourBooking", new
                {
                    tourBookingId = Request.Form["tourBookingId"]
                });
            }
        }

        public async Task<IActionResult> OnPostPackageTableAsync(string command, string returnUrl = null)
        {
            if (command == "Cancel")
            {
                var PackageBookingId = new Guid(Request.Form["packageBookingId"]);

                var packageBooking = await _dbContext.PackageBookings
                    .Where(pb => pb.PackageBookingId == PackageBookingId)
                    .FirstOrDefaultAsync();

                packageBooking.IsCancelled = true;

                await _dbContext.SaveChangesAsync();

                return RedirectToPage("/ViewBookings");
            }
            else
            {
                return RedirectToPage("/EditPackageBooking", new
                {
                    packageBookingId = Request.Form["packageBookingId"]
                });
            }
        }
    }
}
```

## 5.5 Files for hotel, tour, and package Edit Bookings

### 5.5.1 [`EditHotelBooking.cshtml`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Pages/EditHotelBooking.cshtml)

```csharp
@page
@model EditHotelBookingModel
@{
    ViewData["Title"] = "EditHotelBooking";
}

<h2>Edit Hotel Booking</h2>
<div asp-validation-summary="ModelOnly" class="text-danger" role="alert"></div>
<form id="hotelSearchForm" method="post">
        <input type="hidden" asp-for="EditBooking.HotelBookingId" value="@Model.EditBooking.HotelBookingId" />
        <hr />
        <div class="form-group">
            <label asp-for="EditBooking.CheckInDate">Select check-in date:</label>
            <input asp-for="EditBooking.CheckInDate" type="date" class="form-control" value="@Model.EditBooking.CheckInDate.ToString("yyyy-MM-dd")" />
            <span asp-validation-for="EditBooking.CheckInDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label asp-for="EditBooking.CheckInDate">Select check-out date:</label>
            <input asp-for="EditBooking.CheckOutDate" type="date" class="form-control" value="@Model.EditBooking.CheckOutDate.ToString("yyyy-MM-dd")" />
            <span asp-validation-for="EditBooking.CheckOutDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label>Room type:</label>
            <input type="text" value="@Model.EditBooking.RoomType" disabled/>
        </div>
        <div class="form-group">
            <label>Hotel:</label>
            <input type="text" value="@Model.EditBooking.HotelsList.First().Name" disabled />
        </div>
        <p class="text-danger">@Model.EditBooking.ErrorMessage</p>
        <div class="form-group">
            <input type="submit" class="btn btn-primary" name="command" value="Modify" />
        </div>
</form>
```

### 5.5.2 [`EditHotelBooking.cshtml.cs`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Pages/EditHotelBooking.cshtml.cs)

```csharp
using asp_net_core_web_app_authentication_authorisation.Models;
using asp_net_core_web_app_authentication_authorisation.Services;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.EntityFrameworkCore;
using System.ComponentModel.DataAnnotations;

namespace asp_net_core_web_app_authentication_authorisation.Pages
{
    public class EditHotelBookingModel : PageModel
    {
        [BindProperty]
        public EditBookingModel EditBooking { get; set; }

        private readonly ApplicationDbContext _dbContext;
        private readonly UserManager<ApplicationUser> _userManager;

        public EditHotelBookingModel(ApplicationDbContext dbContext, UserManager<ApplicationUser> userManager)
        {
            EditBooking = new EditBookingModel();
            _dbContext = dbContext;
            _userManager = userManager;
        }

        public class EditBookingModel
        {
            [Required(ErrorMessage = "Please select a check-in date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Check in date")]
            public DateTime CheckInDate { get; set; }

            [Required(ErrorMessage = "Please select a check-out date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Check out date")]
            public DateTime CheckOutDate { get; set; }

            public string RoomType { get; set; }

            public List<Hotel> HotelsList { get; set; } = new List<Hotel>();

            public string HotelBookingId { get; set; }

            public string ErrorMessage { get; set; }
        }

        public async Task<IActionResult> OnGet()
        {
            var HotelBookingIdValue = Request.Query["hotelBookingId"];
            var HotelBookingId = new Guid(HotelBookingIdValue.ToString());

            var hotelBooking = await _dbContext.HotelBookings
                .Where(hb => hb.HotelBookingId == HotelBookingId)
                .Include(hb => hb.Hotel)
                .FirstOrDefaultAsync();

            EditBooking.CheckInDate = hotelBooking.CheckInDate;
            EditBooking.CheckOutDate = hotelBooking.CheckOutDate;
            EditBooking.RoomType = hotelBooking.Hotel.RoomType;
            EditBooking.HotelsList.Add(hotelBooking.Hotel);

            EditBooking.HotelBookingId = HotelBookingIdValue;

            return Page();
        }

        public async Task<IActionResult> OnPostAsync(string returnUrl = null)
        {
            EditBooking.ErrorMessage = null;

            var HotelBookingIdValue = Request.Query["hotelBookingId"];
            var HotelBookingId = new Guid(HotelBookingIdValue.ToString());

            var HotelBooking = await _dbContext.HotelBookings
                .Where(hb => hb.HotelBookingId == HotelBookingId)
                .Include(hb => hb.Hotel)
                .FirstOrDefaultAsync();

            var CurrentUser = await _userManager.GetUserAsync(User);

            var HotelAvailability = await _dbContext.HotelAvailabilities
                .Where(ha =>
                    ha.HotelId == HotelBooking.HotelId &&
                    ha.AvailableFrom <= EditBooking.CheckInDate &&
                    ha.AvailableTo >= EditBooking.CheckOutDate)
                .Select(ha => ha.Hotel)
                .Distinct()
                .ToListAsync();

            if (HotelAvailability.Count == 1)
            {
                HotelBooking.CheckInDate = EditBooking.CheckInDate;
                HotelBooking.CheckOutDate = EditBooking.CheckOutDate;

                _dbContext.HotelBookings.Update(HotelBooking);
                await _dbContext.SaveChangesAsync();

                return RedirectToPage("/Payment", new
                {
                    bookingId = HotelBookingIdValue,
                    bookingType = "hotel"
                });
            }
            else
            {
                EditBooking.ErrorMessage = "Hotels not available for selected dates";

                EditBooking.HotelsList.Add(HotelBooking.Hotel);
                EditBooking.CheckInDate = EditBooking.CheckInDate;
                EditBooking.CheckOutDate = EditBooking.CheckOutDate;
                EditBooking.RoomType = HotelBooking.Hotel.RoomType;

                return Page();
            }
        }
    }
}
```

### 5.5.3 [`EditTourBooking.cshtml`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Pages/EditTourBooking.cshtml)

```csharp
@page
@model EditTourBookingModel
@{
    ViewData["Title"] = "EditTourBooking";
}

<h2>Edit Tour Booking</h2>
<div asp-validation-summary="ModelOnly" class="text-danger" role="alert"></div>
<form id="tourSearchForm" method="post">
        <input type="hidden" asp-for="EditBooking.TourBookingId" value="@Model.EditBooking.TourBookingId" />
        <hr />
        <div class="form-group">
            <label asp-for="EditBooking.TourStartDate">Select start date:</label>
            <input asp-for="EditBooking.TourStartDate" type="date" class="form-control" value="@Model.EditBooking.TourStartDate.ToString("yyyy-MM-dd")" />
            <span asp-validation-for="EditBooking.TourStartDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label asp-for="EditBooking.TourEndDate">Select end date:</label>
            <input asp-for="EditBooking.TourEndDate" type="date" class="form-control" value="@Model.EditBooking.TourEndDate.ToString("yyyy-MM-dd")" />
            <span asp-validation-for="EditBooking.TourEndDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label>Tour:</label>
            <input type="text" value="@Model.EditBooking.ToursList.First().Name" disabled />
        </div>
        <p class="text-danger">@Model.EditBooking.ErrorMessage</p>
        <div class="form-group">
            <input type="submit" class="btn btn-primary" name="command" value="Modify" />
        </div>
</form>
```

### 5.5.4 [`EditTourBooking.cshtml.cs`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Pages/EditTourBooking.cshtml.cs)

```csharp
using asp_net_core_web_app_authentication_authorisation.Models;
using asp_net_core_web_app_authentication_authorisation.Services;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.EntityFrameworkCore;
using System.ComponentModel.DataAnnotations;

namespace asp_net_core_web_app_authentication_authorisation.Pages
{
    public class EditTourBookingModel : PageModel
    {
        [BindProperty]
        public EditBookingModel EditBooking { get; set; }

        private readonly ApplicationDbContext _dbContext;
        private readonly UserManager<ApplicationUser> _userManager;

        public EditTourBookingModel(ApplicationDbContext dbContext, UserManager<ApplicationUser> userManager)
        {
            EditBooking = new EditBookingModel();
            _dbContext = dbContext;
            _userManager = userManager;
        }

        public class EditBookingModel
        {
            [Required(ErrorMessage = "Please select a tour start date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Tour start date")]
            public DateTime TourStartDate { get; set; }

            [Required(ErrorMessage = "Please select a tour end date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Tour end date")]
            public DateTime TourEndDate { get; set; }

            public List<Tour> ToursList { get; set; } = new List<Tour>();

            public string TourBookingId { get; set; }

            public string ErrorMessage { get; set; }
        }

        public async Task<IActionResult> OnGet()
        {
            var TourBookingIdValue = Request.Query["tourBookingId"];
            var TourBookingId = new Guid(TourBookingIdValue.ToString());

            var tourBooking = await _dbContext.TourBookings
                .Where(hb => hb.TourBookingId == TourBookingId)
                .Include(hb => hb.Tour)
                .FirstOrDefaultAsync();

            EditBooking.TourStartDate = tourBooking.TourStartDate;
            EditBooking.TourEndDate = tourBooking.TourEndDate;
            EditBooking.ToursList.Add(tourBooking.Tour);

            EditBooking.TourBookingId = TourBookingIdValue;

            return Page();
        }

        public async Task<IActionResult> OnPostAsync(string returnUrl = null)
        {
            EditBooking.ErrorMessage = null;

            var TourBookingIdValue = Request.Query["tourBookingId"];
            var TourBookingId = new Guid(TourBookingIdValue.ToString());

            var TourBooking = await _dbContext.TourBookings
                .Where(hb => hb.TourBookingId == TourBookingId)
                .Include(hb => hb.Tour)
                .FirstOrDefaultAsync();

            var CurrentUser = await _userManager.GetUserAsync(User);

            var TourAvailability = await _dbContext.TourAvailabilities
                .Where(ta =>
                    ta.TourId == TourBooking.TourId &&
                    ta.AvailableFrom <= EditBooking.TourStartDate &&
                    ta.AvailableTo >= EditBooking.TourEndDate)
                .Select(ha => ha.Tour)
                .Distinct()
                .ToListAsync();

            if (TourAvailability.Count == 1)
            {
                TourBooking.TourStartDate = EditBooking.TourStartDate;
                TourBooking.TourEndDate = EditBooking.TourEndDate;

                _dbContext.TourBookings.Update(TourBooking);
                await _dbContext.SaveChangesAsync();

                return RedirectToPage("/Payment", new
                {
                    bookingId = TourBookingIdValue,
                    bookingType = "tour"
                });
            }
            else
            {
                EditBooking.ErrorMessage = "Tours not available for selected dates";

                EditBooking.ToursList.Add(TourBooking.Tour);
                EditBooking.TourStartDate = EditBooking.TourStartDate;
                EditBooking.TourEndDate = EditBooking.TourEndDate;

                return Page();
            }
        }
    }
}
```

### 5.5.5 [`EditPackageBooking.cshtml`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Pages/EditPackageBooking.cshtml)

```csharp
@page
@model EditPackageBookingModel
@{
    ViewData["Title"] = "EditPackageBooking";
}

<h2>Edit Package Booking</h2>
<div asp-validation-summary="ModelOnly" class="text-danger" role="alert"></div>
<form id="packageBookForm" method="post">
    <input type="hidden" asp-for="EditBooking.PackageBookingId" value="@Model.EditBooking.PackageBookingId" />
        <hr />
        <div class="form-group">
            <label asp-for="EditBooking.CheckInDate">Select check-in date:</label>
            <input asp-for="EditBooking.CheckInDate" type="date" class="form-control" value="@Model.EditBooking.CheckInDate.ToString("yyyy-MM-dd")" />
            <span asp-validation-for="EditBooking.CheckInDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label asp-for="EditBooking.CheckInDate">Select check-out date:</label>
            <input asp-for="EditBooking.CheckOutDate" type="date" class="form-control" value="@Model.EditBooking.CheckOutDate.ToString("yyyy-MM-dd")" />
            <span asp-validation-for="EditBooking.CheckOutDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label>Room type:</label>
            <input type="text" value="@Model.EditBooking.RoomType" disabled/>
        </div>
        <div class="form-group">
            <label>Hotel:</label>
            <input type="text" value="@Model.EditBooking.HotelsList.First().Name" disabled />
        </div>
        <hr />
        <div class="form-group">
            <label asp-for="EditBooking.TourStartDate">Select start date:</label>
            <input asp-for="EditBooking.TourStartDate" type="date" class="form-control" value="@Model.EditBooking.TourStartDate.ToString("yyyy-MM-dd")" />
            <span asp-validation-for="EditBooking.TourStartDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label asp-for="EditBooking.TourEndDate">Select end date:</label>
            <input asp-for="EditBooking.TourEndDate" type="date" class="form-control" value="@Model.EditBooking.TourEndDate.ToString("yyyy-MM-dd")" />
            <span asp-validation-for="EditBooking.TourEndDate" class="text-danger"></span>
        </div>
        <div class="form-group">
            <label>Tour:</label>
            <input type="text" value="@Model.EditBooking.ToursList.First().Name" disabled />
        </div>
        <p class="text-danger">@Model.EditBooking.ErrorMessage</p>
        <div class="form-group">
            <input type="submit" class="btn btn-primary" name="command" value="Modify" />
        </div>
</form>
```

### 5.5.6 [`EditPackageBooking.cshtml.cs`](https://github.com/iArcanic/pacific-tours-ccse-cw1/blob/main/Pages/EditPackageBooking.cshtml.cs)

```csharp
using asp_net_core_web_app_authentication_authorisation.Models;
using asp_net_core_web_app_authentication_authorisation.Services;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.EntityFrameworkCore;
using System.ComponentModel.DataAnnotations;

namespace asp_net_core_web_app_authentication_authorisation.Pages
{
    public class EditPackageBookingModel : PageModel
    {
        [BindProperty]
        public EditBookingModel EditBooking { get; set; }

        private readonly ApplicationDbContext _dbContext;
        private readonly UserManager<ApplicationUser> _userManager;

        public EditPackageBookingModel(ApplicationDbContext dbContext, UserManager<ApplicationUser> userManager)
        {
            EditBooking = new EditBookingModel();
            _dbContext = dbContext;
            _userManager = userManager;
        }

        public class EditBookingModel
        {
            public string PackageBookingId { get; set; }

            [Required(ErrorMessage = "Please select a check-in date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Check in date")]
            public DateTime CheckInDate { get; set; }

            [Required(ErrorMessage = "Please select a check-out date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Check out date")]
            public DateTime CheckOutDate { get; set; }

            [Required(ErrorMessage = "Please select a room type")]
            [DataType(DataType.Text)]
            [Display(Name = "Room type")]
            public string RoomType { get; set; }

            public List<SelectListItem> RoomTypes { get; set; } = new List<SelectListItem>
            {
                new SelectListItem
                {
                    Value = "single",
                    Text = "Single"
                },
                new SelectListItem
                {
                    Value = "double",
                    Text = "Double"
                },
                new SelectListItem
                {
                    Value = "family suite",
                    Text = "Family Suite"
                }
            };

            public List<Hotel> HotelsList { get; set; } = new List<Hotel>();

            [Required(ErrorMessage = "Please select a tour start date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Tour start date")]
            public DateTime TourStartDate { get; set; }

            [Required(ErrorMessage = "Please select a tour end date")]
            [DataType(DataType.DateTime)]
            [Display(Name = "Tour end date")]
            public DateTime TourEndDate { get; set; }

            public List<Tour> ToursList { get; set; } = new List<Tour>();

            public string ErrorMessage { get; set; }
        }

        public async Task<IActionResult> OnGet()
        {
            var PackageBookingIdValue = Request.Query["packageBookingId"];
            var PackageBookingId = new Guid(PackageBookingIdValue.ToString());

            var packageBooking = await _dbContext.PackageBookings
                .Where(pb => pb.PackageBookingId == PackageBookingId)
                .Include(pb => pb.Hotel)
                .Include(pb => pb.Tour)
                .FirstOrDefaultAsync();

            EditBooking.CheckInDate = packageBooking.CheckInDate;
            EditBooking.CheckOutDate = packageBooking.CheckOutDate;
            EditBooking.RoomType = packageBooking.Hotel.RoomType;
            EditBooking.HotelsList.Add(packageBooking.Hotel);

            EditBooking.TourStartDate = packageBooking.TourStartDate;
            EditBooking.TourEndDate = packageBooking.TourEndDate;
            EditBooking.ToursList.Add(packageBooking.Tour);

            EditBooking.PackageBookingId = PackageBookingIdValue;

            return Page();
        }

        public async Task<IActionResult> OnPostAsync()
        {
            EditBooking.ErrorMessage = null;

            var PackageBookingIdValue = Request.Query["packageBookingId"];
            var PackageBookingId = new Guid(PackageBookingIdValue.ToString());

            var packageBooking = await _dbContext.PackageBookings
                .Where(pb => pb.PackageBookingId == PackageBookingId)
                .Include(pb => pb.Hotel)
                .Include(pb => pb.Tour)
                .FirstOrDefaultAsync();

            var CurrentUser = await _userManager.GetUserAsync(User);

            var hotelAvailability = await _dbContext.HotelAvailabilities
                .Where(ha =>
                    ha.HotelId == packageBooking.HotelId &&
                    ha.AvailableFrom <= EditBooking.CheckInDate &&
                    ha.AvailableTo >= EditBooking.CheckOutDate)
                .Select(ha => ha.Hotel)
                .Distinct()
                .ToListAsync();

            var tourAvailability = await _dbContext.TourAvailabilities
                .Where(ta =>
                    ta.TourId == packageBooking.TourId &&
                    ta.AvailableFrom <= EditBooking.TourStartDate &&
                    ta.AvailableTo >= EditBooking.TourEndDate)
                .Select(ha => ha.Tour)
                .Distinct()
                .ToListAsync();

            if (hotelAvailability.Count == 1 && tourAvailability.Count == 1)
            {
                packageBooking.CheckInDate = EditBooking.CheckInDate;
                packageBooking.CheckOutDate = EditBooking.CheckOutDate;

                packageBooking.TourStartDate = EditBooking.TourStartDate;
                packageBooking.TourEndDate = EditBooking.TourEndDate;

                _dbContext.PackageBookings.Update(packageBooking);
                await _dbContext.SaveChangesAsync();

                return RedirectToPage("/Payment", new
                {
                    bookingId = PackageBookingIdValue,
                    bookingType = "package"
                });
            }
            else
            {
                EditBooking.ErrorMessage = "Hotels and/or Tours not available for selected dates";

                EditBooking.HotelsList.Add(packageBooking.Hotel);
                EditBooking.CheckInDate = EditBooking.CheckInDate;
                EditBooking.CheckOutDate = EditBooking.CheckOutDate;
                EditBooking.RoomType = packageBooking.Hotel.RoomType;

                EditBooking.ToursList.Add(packageBooking.Tour);
                EditBooking.TourStartDate = EditBooking.TourStartDate;
                EditBooking.TourEndDate = EditBooking.TourEndDate;

                return Page();
            }
        }
    }
}
```

# 6 References
