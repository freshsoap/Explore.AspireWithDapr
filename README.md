# Explore.AspireDapr

This is a codespace that runs the [AspireWithDapr](https://github.com/dotnet/aspire-samples/tree/main/samples/AspireWithDapr) sample.

Absolutely nothing clever was done to get this to work in codespaces, aside from:
- disabling `https` in the sample
- setup Dapr and Aspire in the devcontainer.json

## Why Bother

- https://anthonysimmon.com/dotnet-aspire-best-way-to-experiment-dapr-local-dev/

## TLDR: To Run

Yeah - I've overexplained this, but here is instructions

1. Open this repo as a codespace
2. Start dashboard
3. Login
4. Open frontend
5. Marvel at the awesomeness
6. Eventually get https to work - but not today

### Open this repo as a codespace

On this repo screen
1. goto code button > codespaces > open as codespace
2. It doesnt seem to need extra codespace ram or cpus to run Dapr/Docker

### Start dashboard

Start the dashboard when inside the codespace terminal:  
```bash
# goto solution directory
cd ./AspireWithDapr/
# start dashboard 
dotnet run --project AspireWithDapr.AppHost
```

### Login

1. Get the token for auth:  
   `Login to the dashboard at http://localhost:15278/login?t=1234blahblahblah5678token` shows in console
   - copy `/login?t=1234blahblahblah5678token`
2. Goto the Aspire dashboard
   - goto the Ports tab in vscode 
   - open the url for port 15278 in Browser
   - it _should_ be named `http-Aspire-AppHost(15278)` from the devcontainer
3. Login to the Aspire dashboard
   - Add the `/login?t=tokenblah` to the url
   - Yay! Dashboard shows!
   - Goto dashboard > Resources 
     - all the resources should be running and the webfrontend should show at port `5018`
     - TBH, this is pretty cool

### Open frontend

5. Open the Blazor web frontend
   - Goto the ports tab in vscode 
   - open the url for port 5018 in browser
   - it should be named `http-webfrontend(5018)` from the devcontainer
   - Yay! blazor webfrontend shows!

------------------------------------

Ignore everything after here - its my scratch notes when getting this to work

## Explore setup

Checkout the devcontainer - it sets up docker and dapr

## Aspire with Dapr on Codespaces - Repo Setup

https://learn.microsoft.com/en-us/dotnet/aspire/get-started/build-your-first-aspire-app?tabs=dotnet-cli

1. Create a Github repo
2. Get the AspireWithDapr sample from 
   - https://github.com/dotnet/aspire-samples/tree/main/samples/AspireWithDapr
3. Open your repo in a browser as https://github.dev/YourRepo
4. drag/drop the AspireWithDapr folder in and commit
5. Open this repo as a codespace

## SUCCESS WITH HTTP!!

The copespace works out of box if running the AppHost as http, not Https

### Set Https

1. goto the AppHost project > Properties > launchSettings.json
2. comment out the entire https profile
3. add `"ASPIRE_ALLOW_UNSECURED_TRANSPORT": "true"` to the http profile
   - this is easy to find; dotnet run fails with this error

### Setup from `.devcontainer.json`

If this isnt successful, manual steps are below
1. c#, docker, dapr and aspire _should_ be features in the `.devcontainer.json`
2. Setup Ports - This _should_ be done by the `.devcontainer.json`
3. Init Dapr - this _should_ be done by the `post-create.sh`

I need to get it to work with https... 

## SCRATCH NOTES

The manual installations should be covered by the `.devcontainer.json`

### AspireWithDapr Sample

Get the AspireWithDapr sample from 
- https://github.com/dotnet/aspire-samples
- https://github.com/dotnet/aspire-samples/tree/main/samples/AspireWithDapr
- https://learn.microsoft.com/en-us/samples/browse/?expanded=dotnet&products=dotnet-aspire

### manually Install Aspire in a codespace

Open in a default codespace - it didnt seem to need extra grunt for the docker images/dapr blah

https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/setup-tooling?tabs=dotnet-cli%2Cwindows#install-net-aspire

```bash
dotnet workload update
dotnet workload install aspire
dotnet workload list
```

cd to AppHost proj

```bash
dotnet run
```

### Manually Install Dapr

https://docs.dapr.io/getting-started/install-dapr-cli/

```bash
wget -q https://raw.githubusercontent.com/dapr/cli/master/install/install.sh -O - | /bin/bash
dapr init
```

### Setup ports

1. goto ports tab in vscode and add `15278` for the dashboard, `5018` for the blazor page
2. leave visiblity as Private
3. open the browser on the forwarded address
4. Dashboard
   - 15278 comes from the terminal when `dotnet run`ing
5. webfrontend
   - 5018 comes from the Resources page on the running dashboard
     - it still uses localhost so you have to port-forward the codespace url as described above

## TROUBLESHOOTING SETUP - these didnt really work until setting the dashboaed as just HTTP

```bash
cd ./ToTheProjFolder
dotnet run
```

Terminal shows

```txt
Login to the dashboard at https://localhost:17267/login?t=1111111111111111111111cccccccc
```
on Prts tab, make port 17267 visible
/login?t=blah and try it at the codespace url

Still getting CSP errors

added this just before each `builder.Build();`:
```csharp
builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders =
        ForwardedHeaders.XForwardedFor
        | ForwardedHeaders.XForwardedProto
        | ForwardedHeaders.XForwardedHost;
});
```

Made no difference

### Cert

Looking at https://aka.ms/aspnet/https-trust-dev-cert
dotnet dev-certs https --trust

And lets try this on a not-firefox browser...
I made port public and am running on private chrome

Now lets disable 'app.UseHttpsRedirection();` and make all ports `http`
and make all ports `https`?

FYI - port on 9411 is Zipkin
looks like 6379 is Redis

AppHost LaunchSettings shows port 17267 as https

### Set to http

SUCCESS!!

- Goto AppHost Launchsettings and comment out https config
- add `"ASPIRE_ALLOW_UNSECURED_TRANSPORT": "true"` to its environment variables
