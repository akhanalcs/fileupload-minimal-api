# fileupload-minimal-api
Example repo to upload files to a Minimal API in `.NET 8`.

---

I figured this out after going through [this](https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery?view=aspnetcore-8.0#antiforgery-with-minimal-apis).
As [per the docs](https://learn.microsoft.com/en-us/aspnet/core/release-notes/aspnetcore-7.0?source=recommendations&view=aspnetcore-7.0#file-uploads-using-iformfile-and-iformfilecollection), to be able to upload files you need an Authorization header, a client certificate, or a cookie header.

To satisfy that requirement, I decided to pass *Antiforgery token* with the file upload POST request.

Now my API endpoints looks like this:
```csharp
// Get token
app.MapGet("antiforgery/token", (IAntiforgery forgeryService, HttpContext context) =>
{
    var tokens = forgeryService.GetAndStoreTokens(context);
    var xsrfToken = tokens.RequestToken!;
    return TypedResults.Content(xsrfToken, "text/plain");
});
//.RequireAuthorization(); // In a real world scenario, you'll only give this token to authorized users

// Add my file upload endpoint
app.MapPost("/upload_many", async (IFormFileCollection myFiles) =>
{
    foreach (var file in myFiles)
    {
        // ...
    }

    return TypedResults.Ok("Ayo, I got your files!");
});
```

So I've got 2 endpoints now:

<img src="https://i.stack.imgur.com/eyJsr.png/150" width="300" />

Upload the file
---

**Step 1:**
Get the token from Postman.

<img src="https://i.stack.imgur.com/uvx1k.png/450" width="800" />

**Step 2.1:**
Before making the POST call to Upload endpoint, add the XSRF token you received from Step 1.

<img src="https://i.stack.imgur.com/3oyPX.png/250" width="500" />

**Step 2.2:**
Add the file you want to upload. *pickle.png* in my case.

<img src="https://i.stack.imgur.com/C49zk.png/250" width="400" />

**Step 2.3:**
Hit send. At this point, you'll be able to make a successful call.

<img src="https://i.stack.imgur.com/hYWAR.png/250" width="600" />
