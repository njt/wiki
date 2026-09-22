---
url: https://gist.github.com/njt/23c08df43340b155bc4a706d1aaaa937
date_fetched: 2026-08-07
---

# ytx: ASP.NET Core Authentication - The Dirty Details - Chris Klug - NDC Copenhagen 2026 — NDC Conferences

Source gist: https://gist.github.com/njt/23c08df43340b155bc4a706d1aaaa937



======================================================================
## FILE: 20260807-asp-net-core-authentication-the-dirty-details-chris-klug-ndc-copenhagen-2026.md
======================================================================

### Key points
- **The authentication system is deceptively simple.** Despite being rewritten multiple times, it’s now “quite beautiful” — built entirely on the `IAuthenticationHandler` interface (four methods: `InitializeAsync`, `AuthenticateAsync`, `ChallengeAsync`, `ForbidAsync`) and a central `AuthenticationOptions` object that acts as the registry of all schemes.
- **Authentication ≠ sign‑in.** Most schemes only prove identity; session creation (sign‑in) is almost always delegated to cookie authentication. That’s why `SignInAsync` and `SignOutAsync` are separate interfaces (`IAuthenticationSignInHandler`, `IAuthenticationSignOutHandler`) — and why `SignOutHandler` comes before `SignInHandler` in the hierarchy (remote providers often need to redirect on sign‑out but never handle sign‑in).
- **The `AuthenticationProperties` object is the state‑carrier.** Its `Items` dictionary is serialized (encrypted, base64‑encoded) and passed around — e.g., as the `state` parameter in OpenID Connect. The `Parameters` dictionary is `[JsonIgnore]` and only for in‑process hand‑offs. This is how correlation IDs and scheme‑ownership markers survive round‑trips.
- **Forwarding is built into the base class.** `AuthenticationHandler<TOptions>` checks `ForwardAuthenticate`, `ForwardChallenge`, `ForwardForbid`, `ForwardDefault`, and a `ForwardDefaultSelector` func — if any are set, the handler delegates to another scheme. This is how policy schemes and dynamic scheme selection work.
- **Remote authentication handlers are the most likely custom implementation.** `RemoteAuthenticationHandler<TOptions>` adds callback path handling, backchannel HTTP client, correlation ID CSRF protection (a cookie containing the literal string `N`), and a built‑in flow that validates the correlation, raises events, and hands the principal to the sign‑in scheme.
- **Policy schemes (`AddPolicyScheme`) let you route to different handlers based on context.** Using `ForwardDefaultSelector`, you can inspect the request (path, host, user‑agent) and choose a scheme dynamically — useful for multi‑tenancy or API‑vs‑browser branching.
- **Claims transformation runs on every request — including static files.** `IClaimsTransformation.TransformAsync` is called after authentication on every incoming request, so database calls inside it are a performance disaster.
- **You can wrap existing handlers by replacing `IAuthenticationHandlerProvider`.** The speaker demonstrated a decorator that intercepts the cookie handler and returns 401/403 for API endpoints instead of a redirect, without touching the original handler’s code.

### Pithy and provocative quotes
- “Simplicity does not precede complexity, but follows it.” — opening the talk after discovering the authentication internals are actually clean.
- “The authentication system in ASP NET Core has been rewritten a couple of times. Apparently it’s actually quite beautiful at this point. There are no ugly dirty details, there are no nooks and crannies.”
- “I’m the chief over engineering officer and I believe in that you need to over engineer everything.”
- “The contents of the cookie is a capital N that’s it.” — describing the correlation cookie used for CSRF protection in remote handlers.
- “This is not a thing that runs when you Sign in. This is a thing that runs on every incoming request and if you’re unlucky, it runs on Every incoming CSS JavaScript image. Whatever request comes in, it will run this.” — on `IClaimsTransformation`.
- “Was this overly complicated to solve something that you can do with cookie authentication options? Yes. Was it fun? Yes.” — after showing the API‑aware auth decorator.
- “If you ever say you’ve got it from me, I will hunt you down.” — about the demo facial‑recognition handler’s in‑memory session store.

### Tools, practices, and methodologies
- **`IAuthenticationHandler`** — The core interface with four methods (`InitializeAsync`, `AuthenticateAsync`, `ChallengeAsync`, `ForbidAsync`). Implement this directly if you don’t want to inherit Microsoft’s base classes (but then you must manually add your scheme to `AuthenticationOptions`).
- **`AuthenticationHandler<TOptions>`** — Base class that provides forwarding logic, option resolution via named `IOptionsMonitor`, time provider setup, and event initialization. Override `HandleAuthenticateAsync` (abstract) and optionally `HandleChallengeAsync`/`HandleForbiddenAsync`.
- **`RemoteAuthenticationHandler<TOptions>`** — For external identity providers. Adds callback path handling, correlation ID CSRF protection (generate random bytes, store in `Items`, issue a cookie with value `N`, validate on return), backchannel HTTP client, and a built‑in flow that calls `HandleRemoteAuthenticateAsync` (abstract) and then signs in via the configured `SignInScheme`.
- **`AuthenticationProperties`** — Use `Items` dictionary for any data that must survive a round‑trip (serialized, encrypted). Use `Parameters` for in‑process hand‑offs. The `Items` dictionary is where you stash the scheme name during sign‑in so the handler can later verify ownership.
- **Policy schemes (`AddPolicyScheme`)** — Register a scheme with no handler of its own; use `ForwardDefaultSelector` to dynamically pick another scheme based on `HttpContext`. Example: `if (context.Request.Path.StartsWithSegments("/api")) return "Bearer"; else return "Cookies";`
- **Correlation ID pattern** — Built into `RemoteAuthenticationHandler`. Generates 32 random bytes, stores in `AuthenticationProperties.Items` under `".xsrf"`, issues a cookie named `".Correlation.{scheme}.{correlationId}"` with value `"N"`. On callback, validates that the cookie exists, matches, and then deletes it.
- **Handler decoration via `IAuthenticationHandlerProvider`** — Replace the default provider in DI, intercept the creation of a specific handler, wrap it in your own decorator that implements the same interfaces, and return the decorator. Allows you to modify behavior (e.g., return 401 instead of redirect for API calls) without touching the original handler.
- **Embedded resources for handler UI** — The speaker served HTML login pages directly from the handler by reading embedded resources and writing them to the response. He called it a “shit idea” but it demonstrates how `IAuthenticationRequestHandler` can completely take over the pipeline.
- **`RequireAuthenticatedSignIn`** — Defaults to `true` now; prevents signing in with a non‑authenticated `ClaimsPrincipal` (one missing an `AuthenticationType`). Previously it silently failed.

### Unanswered questions and omissions
- **The authentication middleware pipeline is hand‑waved.** The speaker notes that `UseAuthentication` is injected automatically if you call `AddAuthentication` and says “whole other talk” — the exact ordering, interaction with other middleware, and the “original path” tracking are never explained.
- **No discussion of token storage, refresh, or revocation.** The remote handler demo fakes an OIDC flow but skips real‑world concerns like secure token storage, refresh token rotation, or backchannel logout.
- **The demo handlers are deliberately insecure.** In‑memory cache, no load balancing, no proper session management — the speaker explicitly warns against production use, but doesn’t contrast with what a production‑grade custom handler would require.
- **How to test custom authentication handlers is not covered.** No mention of unit testing or integration testing strategies for handlers, especially those that issue challenges or interact with external services.
- **The `AddScheme` limitation is mentioned but not solved.** The speaker points out that `AddScheme` overloads only work if you inherit from Microsoft’s base classes; if you implement `IAuthenticationHandler` directly, you must manually modify `AuthenticationOptions`. He doesn’t show how to do that cleanly or why Microsoft won’t fix it.
- **Authorization is barely touched.** The talk focuses on authentication; the interplay with authorization policies, requirements, and how the authenticated principal flows into authorization is left out.
- **Multi‑tenancy and dynamic scheme registration at runtime are not explored.** The policy scheme example is static; real‑world scenarios where tenants are added at runtime and schemes must be registered dynamically are not addressed.
- **Error handling and logging in production handlers are glossed over.** The remote failure event is mentioned, but robust error recovery, logging, and user‑friendly error pages are not discussed.


======================================================================
## FILE: transcript.md
======================================================================

# ASP.NET Core Authentication - The Dirty Details - Chris Klug - NDC Copenhagen 2026

- **Channel:** NDC Conferences
- **URL:** https://www.youtube.com/watch?v=vZC9wLWU3i0
- **Duration:** 1h 3m 29s
- **Transcribed:** 2026-08-07

---

I see no more people coming in and I have more stuff to cover than I can do in an hour. So I'm gonna get started. Haha. Thank you. Hello everyone.

You're in the room for ASP NET Core Authentication. Kind of deep issue dive, kind of looking at how it actually works. I called it the Dirty Details because I when I came up with the talk I was like, there's gonna be so many ugly corners in there and nitty gritty details in the nooks and crannies of authentication that it's going to be awesome to look at those ugly parts. We'll come back to that. But.

My name is Chris Klug. I work as a software developer slash architect, public speaker slash stuff at a company called Active Solution in Sweden. Been doing software development professionally since 2000 I think. And I don't know, I like talking about things I've learned and spreading knowledge and talking to other people and going to conferences is part of my job. So I get to hang around with cool people and talk about what they are building and learn and get this network of things.

So it's been fun. And the topic for today is as NET Core Authentication. Now as I said, I named the talk the Dirty Details because there's going to be ugly stuff in there, there's going to be weird parts that are hard to understand and everything. And then I started playing with it and diving deep into it, cloned the repo, went through everything and I came down with having to use this quote in my talk. Simplicity does not precede complexity, but follows it.

The authentication system in ASP NET Core has been rewritten a couple of times. Apparently it's actually quite beautiful at this point. There are no ugly dirty details, there are no nooks and crannies. It's just very, very clean. But I still think we can do a talk about it because understanding what's there allows you to go in and do your thing.

Like I don't like this. I want to build my own or I want to make something up and build it. So I still think there's value in looking at it. And yes, I want to mention that all the code that goes on the screen because a lot of code is going to be in PowerPoint. I have not written the code in PowerPoint really, but I have done some of it and color coded it correctly so everything looks like it's in an editor.

But I have also reformatted it so most of the code comes out of the ASP NET Core repo. But there's so much stuff that I didn't want to go through. So I've removed things like null checks and all of that stuff and kept the most important parts in the code. If you look at this and it's like, it doesn't look like that. No, it doesn't.

There. I've made some changes in the formatting, but the idea is still there. And like I said, it's actually quite nice. Now, this is it. This is the foundation of all of ASMIT's core authentication.

It's the I O authentication handler interface. It's all built on that and it's ridiculously simple. There are like four methods you need to implement to get it working. InitializeAsync it takes an authentication scheme, which is basically the name of the scheme that it's connected to and HTTP context, and it allows you to set up your handler so, for example, record things from the hbcontext or whatever you need to do your work. So it's done like that instead of through a constructor parameter.

Then there's an authenticateasync method. As you can see, it doesn't take any input parameters, it just says authenticate. And then since you've got access to the HTTP context, you can then go and figure out however you want to authenticate the user. There's a challenge, async, that happens when, hey, here's a user trying to go in and access a prohibited resource. What do you want to do?

And in this case you get some authentication properties passed in, but other than that, it's basically do what you want, we don't really care. Go and do something. That's it. And for forbid, kind of the same thing. We have an authenticated user trying to reach a resource that he or she is not allowed to read or access.

Do something. That's it. That's the entire thing that is authentication. If you can figure these four things out, you can pretty much build your own handler and you're good to go. Now, that is kind of cool.

But nowhere in that is there actually something that says sign in or sign out, which is kind of a big part of authentication. But the reason for that is that most schemes don't actually implement sign in or sign out. Most people or sorry, most schemes just support authentication because there's a difference between authenticating someone and signing in, Right? Authenticating is proving you are you, whereas signing in is taking, I know that you are you. I am now going to create some form of session to keep track of that you are you.

So that when you come back with each one of your requests, I know that you're still you, right? That last part of creating that session to keep track that you are you isn't handled by more than a few schemes. Like cookie authentication is pretty much the only one that people use. All the other schemes piggyback on cookie authentication for that last part, which is why we don't have that as part of the authentication I authentication handler thing. But it also makes for a weird abstract reflection if you look at it, because there's another interface you can implement, which is IO authentication.

Sign out handler implements IO authentication handler. So sign out is actually there before sign in. And the reason for that is that if you for example, are building an OpenID Connect provider or anything that's remote authenticated, you're very unlikely to handle the session thing, the sign in part. But when the user signs out, you probably want to redirect the user to that other thing to cancel their single sign on session in that end. So it's actually more likely that you implement sign out than you implement sign in.

So that's why we get iauthentication handler and then we get the sign outhandler as the next step. Then you get the sign in handler and that basically gives you the entire surface. So yes, you get, if you want to do a full on authentication handler that supports all the features you need to implement six methods. That's it, that's the entire thing. And then there's this weird last interface called I authentication request handler implements I authentication handler, has a handler request, async method returns bool and that is for third party authentication.

So basically when you redirect someone to Facebook or Entra or Oxtar or whatever to do authentication remotely, they're gonna come back to your application and the handler then needs to basically look at every incoming request to see, hey, is this someone coming back with an authentication thing that I need to handle? If you implement that, you get access to every incoming request and it allows you to say, hey, hey, I wanna handle this request. Just you don't have to do the rest of the pipeline, I'll take over. And that's where the bool comes in. If you return true, pipeline stops running, false, it just keeps going.

I also mentioned these authentication properties, cause you see those, they're used in some places, they're kind of hard to figure out what they are. They, they get passed into two of these methods, or actually it's also into sign in and sign out. And they're just plain PO's. They look like this. And there's two dictionaries.

There's one, items. Items is serializable, which means that it's actually. These properties are generally serialized and stored. When you send someone off to OpenID Connect, it's being used as a state. If you're doing cookie authentication, it gets serialized as part of your sign in.

So here in that dictionary, you can save things that you want to have access to some time later. The parameters 1 is jsonignore, so it's not serialized and it's just for things in process. I want to pass some information to my handler that passes it somewhere else. It's like a way for us to pass things around during authentication. And then we have these extra properties down here, which are pretty much just statically typed accessors for the items dictionary.

So they're not serialized either. Wait, were there two of those? No. Okay, sorry. There was a.

So Jason ignoring those. So if we look at, for example, the OpenID Connect handler that I mentioned, you come into handle challenge async internal, which is obviously like an internal private version of doing the authentication async part or sorry, challenge part, it goes and creates an OpenID Connect message that is going to be passed off to the OPIC connect endpoint. It sets some properties on that, but then it says message state and then it does state data format protect properties. So what happens there is that it basically takes your items dictionary, serializes it to JSON, encrypts it and adds it, or base 64 encodes it and adds it as the state parameter to your request to Facebook enter whatever. Then the person logs in over there and gets authenticated, gets redirected back, and that state parameter is then redirected back from the prior identity provider.

And when you come in to hand remote authenticateasync, which is when somebody's coming back with an authentication like an OpenID Connect token, for example. One of the first things this system does, it goes and reads that message state property and basically decodes it, unencrypts it, and turns it into an authentication property. So we can actually add things there that come back and you'll see why in a second. But it's good to know the way that the authentication properties work. Now, you generally don't go and implement iOS authentication handler.

You generally go and inherit from authentication handler of T options, which is the base class that Microsoft gives us. And it gives us some helping things. One thing it introduces is the idea of T options. So T options is your configuration is the way that you configure your handler to do its work. There's a base class for that called Authentication Scheme Options and it looks like this.

So it basically has a few little properties that is likely to be useful for all handlers, like claims issuer. If you create claims, you're supposed to use that property to set the issuer of the claim for different reasons. You have the forward things in here. Forward default, forward, authenticate, forward, challenge, forward forbidden. These are ways for us to say that hey, I don't want to handle challenge, for example, someone else can do that part, but I will handle some of the other things that can be configured in here.

It has a forward that thing which is kind of cool. It allows you to define do you want to forward this authentication or challenge or whatever to someone else, but in a FUNC form. So it basically gives us the context. So based on the HTTP context we can go and say, oh, since you're logging in from this IP address, I actually want you to use another form of authentication, for example. That's one way to do it.

And then there's this little thing here. They invent this idea of events while using an authentication handler and the user signs in. There are events that we might be interested in looking at. So like parts of the sign in flow that might be interesting. And yes, it's nullable object, not a very useful type to have anywhere.

But the idea here is that when you create your own, like for example the cookie authentication options that inherits from authentication scheme options like that and then it goes and it adds whatever properties is needed for the authentication scheme to do its work and you then do a public new. It's the only place I've kind of seen new being used in any form of production code at all. It always feels like a hack to to use new, but it's done here. So it goes public new cookie authentication events events. And then there are specific events for cookie authentication like on Validate principle, on signing in, on signed in, on redirect, access denied on redirect, all of these things that you might be interested in plugging into as extension points.

Those are those then added to these events and they are added to the options so that your handler can have a look at those and figure out what to do. Other than that it implements all of the methods that you expect, the methods that the I authentication handler expects. So it implements I initializeasync, authenticateasync, challengeasync, forbidasync, all of those. But they all work in kind of the same way. They do something that's required by the base class and then they call a corresponding virtual or abstract method that you need to to or can override.

Like if you want to do something during initialization, you override initialize handlerasync if you want to well, handle authenticateasync well, that's abstract, that's up to you to implement. Because the base class doesn't know how to authenticate anyone, that's up to you completely. So that's abstract. And the other ones are just handle challenge and handle forbidden and you can go in there and change what you want. And the base pass implementation is actually fairly basic.

Initializeasync looks kind of like this. It takes the input parameters and stores those as properties, which means that your class that inherits from this you have access to the authentication scheme and the context, which is very useful. It gets the options. Now the options like the configuration is actually registered in a named way. So see, here you go.

Optionsmonager get scheme name. It means that there is one options registered in DI for every scheme that is being registered. That allows you to register multiple schemes like multiple versions of the same thing. So you can have many OpenID Connect authentication handlers with different configuration in there. And the only difference is that the options instance is just a named instance based on the name.

That's the way it figures out how to do different configuration. It sets up a time provider because apparently for reasons time is really important during authentication. Apparently a security thing to know time properly. It sets up some events, so if you haven't set up the events, it will do it for you. And then it calls that initialize handler async and if you want to do something, you can override that.

That's kind of the way it works. This is one of the more sort of esoteric ones because it's not like here. You can do a lot of different things. But if we look at authenticateasync, for example, all of these authenticateasync for bidasync and challenge or challenge async kind of do the same thing. They start out with looking at the options at the forward authenticate properties like am I supposed.

Is there something in that? If that's null, it looks at the options forward defaultselector saying hey, do you want to figure out where to pass me instead? And if that returns null, it goes and looks at the forward default. And if none of those are set, then it carries on. If any of them are set, they actually go and say hey, HTTP context.

I actually Want you to go and authenticate the user using this other thing over here. I am not responsible. So it basically just forwards that. And all of that forwarding stuff is in that base class. It's not something that the system does for you, but it's something that comes out of the base class.

And if there are no forwarding things, it just calls that that virtual or in this case protected method that you need, sorry, abstract method that you need to implement to do the work. As I said, they are kind of the same. If we go into a challenge, it looks at the forwarding stuff and then it calls the internal virtual method. It's nice enough to do some null coalescing things so you don't have to handle whether or not the properties are null. And then it actually has a default implementation.

So in challenge it just returns a 401 unauthorized. And if we go and look at the forbid kind of same thing handle forwarding call the internal thing and the default implementation is return 403 forbidden. That's what we get out of the base class. Now this only implements I authentication handlers. So if you need to have anything else, Microsoft is obviously nice enough to go and say, hey, we have other things for you.

So if you want to go and implement the sign out handler, then there's a authentication handler of T options. There is a sign in authentication handler of T options and they're pretty much what you expect. It is the previous thing, plus that extra method that gets added for this, for that interface, that's it. And it does the forwarding for you and then calls that handle method. That's it.

The last one is more interesting and that's the one called remote authentication handler. And that is the one that I kind of feel you're most likely to implement because that is what most people do. Because if you do local authentication, it's cookie authentication. That's what we do. We do cookie authentication because it's simple.

Any other thing is generally a remote thing. So let's have a look at that one in a bit more depth. So first of all, it implements the authentication handler of the options plus the I authentication request handler. So it says, hey, I need to plug in and listen to incoming requests on top of being a handler. And then it has its own set of options.

So so it basically has a remote authentication options instead of authentication scheme options. And the reason for that is that it adds some remote specific kind of properties like since you're gonna be remoting some or sending someone off, you're likely going to have to have a callback path where they call come back after having been authenticated. You're probably gonna have to pass that along to the other to say that hey, once you've authenticated a user, return me here. And that's like what kind of URL parameter, what's the name of the URL parameter where you pass that in? Very likely that you're gonna use it gives you a HTTP client by default.

So if you do for example OpenID Connect, you're very likely to do backchannel stuff. So you get a code back from the handler or sorry from the identity provider and then you need to exchange that code and that's done server to server and you do that using the backchannel, which is just an HTTP client sign in scheme. Like I said, remote authentication providers or remote authentication handlers almost always like forwards the sign in part to someone else. So that's in there as well. Save tokens allows you to define whether or not the tokens you get back should be saved inside of your cookie has no function whatsoever, doesn't actually do anything.

The base class doesn't care about it. It's just there because it's very likely to be used and it is used in a lot of OpenID Connect, for example won't allow you to redirect when you sign out unless you pass your original token to the sign out endpoint. Otherwise you won't be allowed to come back after the sign out according to the spec. For example correlation stuff. Because we're sending people off to do remote things and then come back again.

We probably want to make sure that we are not somebody's doing some cross site request forgery stuff on us or anything like that. So we need correlation stuff in here as well pre built for us and then some custom events for this. So for remote things there are these events like hey, we got a ticket, basically we've got a user has signed in or hey, something failed remotely or something happened with access denied. You might want to plug into that. So those are the things and I can see here there's not a whole lot.

It looks pretty much like it did before. Now I am going to deep dive into this even further. I'm sorry but this is the one that I think we should kind of look at because it's kind of interesting. So I'm sorry about that. But if we had a look at this, the important or interesting thing is like okay, it overrides handle authenticateasync because it needs to know how to do authentication and it overrides forbiddenasync.

If we start with the handleauthenticate, this is what it does. It asks the HTTP context to authenticate the user using the sign in scheme because we are not responsible. So hey, you can sort that out, that's not my problem. And when it comes back it just figures out has this worked or not? If it has worked and everything is fine, it carries on.

And then it goes, hey, the problem here is that I might go and ask these cookie authentication scheme to authenticate the user and I get a user back, but is that really a user that I authenticated? Am I responsible for this? And the way that it verifies that is that when you get this answer back from the sign in scheme is that it looks say okay, is there a principle? Basically somebody logged in. If there is, it gets this.auth scheme thing from the authentication properties.items and it gets that and it compares it to the name of this scheme.

So during sign in, when you sign in, I'm going to show you that. But it basically adds this thing to the properties and this allows me to verify that the authenticated user that comes back was actually authenticated by me. Because I'm not supposed to be like stealing other people's authentication and say that I'm responsible for them. So that's why it does that and all. If all of that is fine, it returns success.

Otherwise it goes, hey, no, sorry, I have no idea, I don't know, it's not my problem. No result forbidden. It's kind of interesting. It's the only other one they override and they override it by doing that. So it's a one liner.

It's basically, hey, if somebody needs to be forbidden, then that's not my problem. Let's ask the other thing. So very likely this is going to end up being hey cook your authentication handler. There's a forbidden thing, what do you want to do? And cook your authentication then redirects the user to sign in.

That's kind of how it works. This is the interesting thing. It's the iAuthentication request handler. And that's as I said, it allows you to plug into any request that comes into your server. So if we look at the implementation, here it goes, is this a user coming back from a third party?

And the way it figures that out is that it runs this piece of code here. There's more. It's a bit long, but yeah, should I handle this request? Default implementation options Callback path equals this path. If that is True, then I will handle it.

Otherwise I return false or it will return false as I have nothing to do with this. Run the rest of the pipeline. If that is true, it goes in and it says, hey, handleremote. That's the one that you have to override. So that's the abstract thing that you have to do.

And then that comes back with an auth result and it verifies the auth result. Just making sure. Did everything go well? Was it handled? Did we need to skip it if it wasn't succeeded?

Take the exception and the properties and all of that. And then once it got that, it goes in and says, hey, was there an exception? If there was, it creates a context, it raises the remote failure event to ask you is like, is there anything you want to do? Something went wrong? And then if that is handled or not, Depends.

Sort of shows you what you want to do here. And then finally most of the time it comes back here. All of that is fine. It comes in here, creates a new ticket received context saying, we now have a ticket, we have a user, that ticket is. There it is properties.itemsauthscheme.

so this is where we plug in and say, hey, you are now being authenticated using this scheme. So we can verify that when we do the authentication part raises an event, checks the result and then in most cases it ends up here and says, okay, sweet, everything went fine. Hey, sign in scheme, can you please sign in this user? Here is the principal, here are the properties. And then when you're done with that, I'm just going to redirect to the return URI and then return true.

The other thing it implements is correlation ID and cross site request. Forgery is a problem, right? Because we have people that we shove off and ask them to go and do something somewhere else. And then HTTP stateless. So somebody comes back and says, hey, I was just authenticated.

Here is the information you need to prove that I am authenticated. That's kind of a scary proposition. So to fix that, we use these correlation things and it has that built in, which is kind of nice. So it starts out, generates 32 bytes of random text base 64 encodes it, adds it to once again the authentication properties for items. So, so it's serialized under the XSRF item and then it gets some cookie options.

It creates a cookie or, sorry, it creates a cookie name based on the configured cookie name and the correlation ID as the name. And then it issues a cookie using that name and the contents of the cookie is a capital N that's it. That gets sent off when they go and authenticate and when they come back in again. You call this thing here called validate correlation ID and invalidatecorrelationID. It looks at the items and says, hey, is there an XSRF thing in there?

If not, then you're not validated. If it is, remove it because we don't want it. After this is executed, find the cookie name based on that correlation id, pull it out. If there is no such cookie, you're not authenticated or you're not validated. Otherwise build some cookie options, delete the cookie because we don't need that cookie anymore, and then get the cookie contents.

Verify that the cookie contents is capital N. If it is, the correlation is correct, it's okay, otherwise it's false. So it's a very, very crude way of basically saying we'll take a value, we'll put that value into a cookie and we'll put it into a thing that we know. We'll send you off and when you come back we'll look at the place, the dictionary where we put it, which we know is safe because that was encrypted, and pass the state and we'll compare that with your cookie. If they align, I know that you have been here and you've started the authentication.

If they do not align, then screw you and go somewhere else. We don't need you anymore. That was a lot. Sorry, I say it was a deep dive. There's a lot of interfaces and classes and things.

So can we go and use some handlers now? No, we can't because there's more stuff in here because handlers is only one part. So let's go jump up to an application I built. This application I'm so proud of has a few things that's interesting. Add authentication, Add cookie and use authentication.

That's the way that we plug into authentication. The first thing adds the services cookie, adds the cookie handler. Use authentication, adds some middleware for us to handle actual authentication. And then finally we have a little endpoint down here that writes out the user's name if they're authenticated. That's it.

This thing here, has anyone looked into what this actually does? I like you. You are just as weird as me. I have. Obviously it adds this lots of stuff.

First of all, we can ignore those because those are not interesting. That leaves us with that. And the top one here, authentication service, it uses the scheme provider and the handler provider to basically based on the scheme, it uses the handler scheme provider to Find the authentication scheme based on a name. It uses the handler provider to get the handler and then ask the handler to do the work. Very boring.

This thing here. It gets a scheme based on a string. What's a scheme? A scheme is the name, which is the thing you're searching on a display name and a type. That's it.

Authentication handler provider. Yeah, it uses the other provider to get the scheme, find the handler, the type of the handler, instance that handler for you, and basically returns an instance of that handler. Really boring. Really pretty much useless. Claims transformation allows you to transform claims in your principles when they come in and they get authenticated.

It's a no op implementation. It's really useless. You don't have to care. You can replace it, but by default it literally does nothing. It returns task completed task.

That's it. So let's ignore that. That thing there. Completely useless. It it gets.

I'm not kidding, it is completely useless for you. It returns authentication configuration from I configuration. So basically it allows you to configure authentication using app settings. Nobody does that properly or such, but it gives the testers a way to plug into configuration using a hand like this instead. And it just simplifies things for them to the testing thing.

This is the cool stuff. And you go, Chris, that's a little options object. It's the thing where you set like a little name. No, it's actually. That's the root of all of authentication.

That and the Iauthentication handler is everything you need to know. That thing looks like this and there's some other stuff, but this was the important stuff. It has a default scheme thingamajig. Default scheme, default authentication scheme, default sign in, scheme default sign out. You've all seen this because you've all done that, I assume.

Add authentication options. What schemes should I use? That options is your authentication scheme options. You've also used it here, but you didn't know it because that sets the default authentication scheme property. What here?

Well, it kind of depends. If you only have one other scheme registered that actually turns into that. So yes, you're using it there as well. If you don't like that default scheme thing, there's this weird context set switch thingamajig you can set in your application and then that is disabled. Don't know why, but it's a backwards compatibility thing and I had to try it and it does work.

If you do set that, it stops doing it for you and then authentication breaks down. If you don't set it to give. All right, there Are use cases. There's a weird thing down here. Require authenticated sign in.

Well, wouldn't you always be authenticated when you sign in? Isn't that like a redundant property? No, actually I have run into this because I'm an idiot many times on top of that, which makes me, I don't know, even more stupid by default. Before a few versions ago when you did sign in async and you passed in the principle, if you passed in a principle that wasn't considered authenticated, it would just ignore it and the user wouldn't get authenticated. That's kind of an annoying thing because it just went poof.

Didn't complain, didn't break. The reason for that is that if you don't pass in authentication type when you create your claims identity, it actually is not considered an authenticated user. But Chris, that's obvious with that constructor. Yes, but there are like 16 overloads to that constructor and not all of them take that property. Even so now by default it actually requires you to have a sign in user.

You can turn it off if you want to. These are the kind of interesting things. There's a dictionary or, sorry, there's a new ienumerable of authentication schemes and there's a dictionary schemes. These are all the authentication schemes that are registered in your system. They're read only, can't really do anything about them, but they're there.

And this is why this becomes the foundation of your entire application when it comes to authentication. But it's actually through that Add scheme. Now, Add scheme is something that you've all used, you just don't know it. Because if we back up to Add cookie and we're going to look at what add cookie looks like, it looks like this. First of all, it's an extension method on authentication builder and the first thing it does is that it adds a callback on the cookie authentication options to set some post configuration defaults.

Basically after the thing has run and all the confliction is done, this just goes in and verifies that all the defaults are set so there are no null values and that the slash endings on paths and everything is normalized and it's just cleanup tasks. Then it adds some other validation here to make sure that you're not using an old property that's not being used anymore. Then it calls Builder addscheme. Oh look, Add scheme. It's not the one I just showed you.

This is add scheme on the authentication builder, that scheme and then you pass in your options type and Your handler type looks like this. It's on Authentication Builder. It just passes it in onto another method called addschemehelper, which is a private method helper method. But it passes it into that. And if we go into that now we're four levels deep.

The first thing that happens is that it adds a callback to configure the authentication options. And what does the callback do o add scheme? So now four levels in, what happens when you register a scheme is that it adds it to the authentication options, and then that authentication options is being passed around to all the other services. And it's like the source of truth. It's the repository of all of your handlers.

It's also add only you can't actually remove anything from it, which is a problem in some cases and Microsoft won't fix it even if I ask them. And then it adds your options, your specific options that you have defined based on the authentication scheme name. So one options type per scheme name. And then it adds your handler to the services collection for dependency injection. Now I kind of removed this for have more space.

It also requires you to pass in options of the type of authentication scheme options, and it requires you to pass in an iauthentication handler. So you know that the handler type, which is of type on the scheme is always going to be an iauthentication handler. So that's the way verily verifies it. The add scheme, though, is quite interesting because that doesn't actually take an iauthentication handler. It takes an authentication handler of T, which means that you can't actually use this if you build your own authentication handler.

You cannot go and use any of these overloads because it only works if you inherit from Microsoft base classes. So if you don't want to inherit their base classes, you have to manually go and modify that authentication options thingamajig, which I have a demo of as well. Besides Add Scheme, we also have Add Remote Scheme, which is kind of the same thing. The only difference is that it requires remote authentication options and remote authentication handler. It also calls the Add Remote scheme on Authentication Builder, which the only thing it really does differently is that it adds this thing that verifies that you have set up sign in Scheme because if you haven't set that up, it's going to explode in your face.

And then it calls that Add Scheme Helper. Very boring. The last one though, Add Policy Scheme. Has anyone ever used Add Policy Scheme? Anyone hear of Policy schemes?

They are weird and they're awesome. They are Kind of interesting because you call add policy scheme, it doesn't have a type parameter, it doesn't tell you what scheme to use. It just has a scheme, a display name and some configuration. And the reason for that is that it comes with its own options and handler. But these can be used for things like this.

You can create a policy scheme and say, I have a scheme called PS1. You can point your entire application, every endpoint in your entire application can point and say, I want to be authenticated uses scheme PS1. And then inside of PS1 you can do these things, say, oh, by the way, authentication should be over there and challenge should be used, but that thing over there and forbid should this theme. So that's kind of cool. But it's also what you can do up in add authentication, right?

So it's not very useful. But it becomes useful when it says, oh, I also have this thing here forward default selector that allows you to have a function using that. We can do things like what scheme should I use? Well, if the path starts with/API, I want you to use the bearer scheme. If not cookie scheme or what's your host or you're on localhost, then I want you to authenticate using the local dev scheme.

Otherwise I want you to use the cookie scheme, for example. So it allows us to tweak and we can still point all of our endpoints to one scheme. But that scheme scheme can then delegate the work to other handlers in a smart way or other schemes coming back out here. Again, use authentication. Fairly simple.

All that really does is this extension method. It adds that thing to some app properties that lets the rest of the system know that the authentication middleware has been added. If you don't add authentication middleware yourself, but you call add authentication on your services, that middleware is actually injected by default by the system. So there will always be a use authentication call in your pipeline. Whole other talk, I can probably do an hour on that because I spent way too much time on figuring out the pipeline, but that's why they have that thing.

Then it adds some authentication middleware. What does that middleware do? Well in the invoke set some feature thingy to say that, hey, there's been some authentication stuff. Keep track of the original path, because there are some things you can do in the request pipeline that actually changes the path and base path properties of the HTTP context, which can mess up your authentication stuff. So that changes that.

Then it goes and says, hey, can you get all the schemes that have one of those I authentication request handlers, those that want to plug into the actual incoming request, it gets the iauthenticatehandlerprider that allows me to get hold of actual authentication handlers. Then for each one of those schemes, it goes and says, hey, get that handler on that handler. Call the handler requestasync. If that returns true, stop executing this pipeline. Just return that handler has done something to the HTTP context, which is now done.

So I should just return the response back and not do anything else. If none of them return true, then we carry on. We looked at the scheme providers like is there a default authenticate scheme? If there is, then it goes in here and says, hey, HpContext, can you authenticate the user using that default scheme? And then, oh yeah, it ends up doing this.

In the end, when you call HPContum sign in or authenticateasync, it gets the authentication service, the authentication service has to authenticateasync. It goes and looks at the scheme. If you didn't pass in the scheme, it gets the default scheme. After that it goes and gets the handler for that scheme. If there is no such handler, blow up, ask that handler to do its work.

Like I said, the handler. That little interface with four methods is everything you need. All the other stuff is just plumbing to call these four methods. So it calls the handler, hey, can you authenticate if it's successful, set the. Well, if you have explains transformation, run that and then return success and return that result.

And when you come back here, if that is successful, HTTPContext user is set to that principle and now the user is authenticated. And then if it's successful, it also adds some features. Because this is kind of interesting. Some things you build will not have access to ASP NET cores internal stuff, but it allows you to go and pull out features. So if you build a class library, you don't have to like attach yourself to ASP NET core.

You just need to know how to get features. And then from that on, so you don't need access to HTTP context, you just need that featured array. And then based on that you can figure out who's logged in and things like that. And then finally it runs the rest of the pipeline. So with that it basically means that at that point in your pipeline you're not authenticated and at that point you might be authenticated.

Now it doesn't work properly as that. It also needs these little login and logout and all they kind of do is they call hpcontext sign in async and hpcontext Sign out async and all of those things that you use to do that are extension methods on HPContum. And if we look at AuthenticateAsyncFun, for example, that we just saw the service for when you call this. And all of these work in the same way. When you call this it the HTTP context goes and gets the authentication service and then the authentication service calls authenticateasync on that service.

The authentication service goes and gets the scheme and then from the scheme gets the handler and then calls authenticateasync on that handler. So once again it is dual. Just wrapping around your iauthentication handler. That's the end goal. And I have demos, So I have 21 minutes of trying to blow you away with awesome authentication demos, which is going to be interesting.

So I have this simple little application. Yes, there is an Aspire project in there because I host some other stuff. But in here it's not a whole lot to see. Is that big? That is not big enough for you in the back?

There's no way that you can see that. Can you see that? Okay, cool. So it's fairly simple. It doesn't do very much.

It just. It adds some service defaults for buyer razor pages authorization. I just need authorization for reasons. It adds authentication, it adds cookie. Some static files use authentication.

There's also some things here that if you want to play around with this, all the code is available on GitHub. Afterwards you want to these just log out the authenticated users with that prefix so you can see where they're logged in and things like that. User authorization map, razor pages, two endpoints, sign in and sign out. And when you call those they just do a context of challenge and context of sign in. So we can sign in or sign out.

Redirecting back to slash. And then there is a endpoint called/API me and it writes out your name and it requires authorization. That's the entire thing. Not very interesting. But I have built an authentication provider that looks like this.

Oops, it doesn't look like this. It's added like this. Can I get that? There we go. So facial recognition authentication.

This sounds interesting. It does, yeah. I'm just gonna make that my facial. So all your authentication schemes do come with one of these default classes as well. So authentication scheme name like facial recognition authentication defaults.

And then in there you can always find the default scheme name, which in my case here is facial recognition. So I've added that. Let's see how that works. Now I need a thing because I'M going to show you how insecure this is. See if this works.

Come on, spin up. It takes a little while. There are containers involved and AI, because apparently you're not allowed to do a talk today in today's world without having AI. So I added AI, which means that I downloaded a 1.8 gig image or four images that you can download and run. They're really cool.

So here it is. I have an AUTH demo in here. It's very, very boring. We can do things like call API. And it basically says, hey, I got a redirect back when I tried to call the API because you're not logged in.

Sweet. Sign in. Now this goes over and says, requesting camera. Yep. I'm just going to pull up a thing.

I'm going to pull up this thing here. I'm gonna get that picture. We're gonna do. Actually, I can't. My wife is in that picture.

So that's not gonna work. We'll do that, is it not? Oh, there it is. Like that. And then after you've shown your face and it proved that you were you, it goes, hey, you need a PIN code as well because we can't trust your face.

Okay, so we're gonna go clown, poop, unicorn, taco. Jason Momoa. I am now jason@smolderinglooks.com now, this is kind of cool, right? So first of all, it shows you exactly how insecure facial recognition is, because I just managed to log in as Jason Momoa with my phone. That is why most computers in the world don't support Windows hello.

Because Windows hello doesn't use your camera like that. It uses IR and a bunch of other things to actually do a proper facial recognition. But good enough. Now how did I implement that? How did I get facial recognition running as an authentication provider?

And also, by the way, I didn't show that. But you weren't redirected anywhere. It's on the actual site and it's all in one place. Add facial recognition like this. So if we go into this thing, it's just an extension method.

Here's another extension method. It calls this overload here. It adds the HP context accessor so I can access the HPContum because I need that. It calls add data protection because, oh yeah, I forgot to mention that it also implements obviously sign in and sign out as well, because I didn't want to rely on cookie authentication. So I actually handled the entire session thing as well using data protection API and cookies and stuff.

It's also a shit implementation. Please not use it in any way, shape or form. And if you ever say you've got it from me, I will hunt you down. And yeah, so I add data protection, I add the memory cache because obviously the way that you keep track of people is an in memory cache that only works on my machine. Why would you load balance anything much much simpler based on that?

How many of you have worked with classical asp? You all remember how you set up a session and you set a session ID and then you've pulled the user Ta da. This is the modern version of that do not use demo. Then I add a little singleton here which is basically a user repository. Well, it adds a options monitor thing.

I get an options monitor thing based on this game and the user repo and it's basically just a way to keep track of my users. Then I add a little flow session service which keeps track of the flow through the three step authentication process that I have. So it adds an encrypted code cookie that gets set up at the beginning and then remembers who you were after the facial recognition and then so on. So it's just a little helper. It adds these facial recognition authentication options with the scheme and then I call builder addscheme manually like this and that internally calls that addscheme helper and all of that stuff.

And the post configuration stuff for the authentication scheme is kind of what cookie authentication does as well. So in the post configuration it basically just makes sure that the the endpoint prefix is trimmed so there's no slash at the end. And it just makes sure that there's string comparison. Like all of these things are set up so that they are ready to use if you forget to configure them in the way. And it uses some API stuff to access the facial recognition thing I'm running on my machine.

But the important thing is obviously inside this handler thing here. Now Chris, did you take this beyond what you were supposed to? Yeah, my title on my first slide is oh, that wasn't. What I want to do is over engineer is chief over engineering officer and I believe in that you need to over engineer everything. Here we go.

So if we scroll, what I want to do was that still getting used to a Mac. So it's assigned in authentication handler of T. It implements I authentication request handler as well because I want to handle incoming requests in the initialize handler async. I just set up some things, nothing interesting. It just figures out login, log, pin path and fail path and then it creates a little service thing to get hold of the thing that talks to my facial recognition API handle requestasync in here it goes in and looks at any incoming request and says oh, is it a get or post?

And is to pass my login path, my PIN path or my fail path. And then depending on what it is, it calls one of these little handle login, get login, post, log pinget. All they do is they actually go away, call this thing here and inside here it goes ahead and it pulls out some stuff out of my assembly. So in the end, if we go over here, I think it is into facial recognition. There are some pages and in here are some HTML pages for the login screen.

For example, this is all HTML for the login screen that is set as an embedded resource. So I basically get the embedded resource, get that string and I stream it back. So I've just created my own mini static web server thingy inside of an authentication handler. Shit idea. But it works, not a problem.

You will not be able to modify the looks or anything like that, but who cares? It could probably be distorted in some way. So that's the way that you get the little thing back, the ui. And then once you've been authenticated, your face has been handled, it gets passed in here as a post and I just pull out the base 64 encoded string version of your image, yada yada, send it off to the authentication service to verify the thing. But that's not really interesting.

But it just goes through this flow by serving up its own files using the iauthentication request handler. And it takes care of everything like that. And then in the handle sign in, for example, it goes in. And what it really does is that it sets a session cookie, basically a cookie with a random string inside of it, encrypted. And then that is an ID that I use in the, the.

The memory cache that I'm using. So basically I take your. When you come back and you're authenticated, I take your principle and I put it into the memory cache with a randomly generated id. Might be a guid not going to tell you because it's implementation details. And then I take that GUID and I encrypt that and put it into my cookie and I give you that cookie and then when you come back and you do authenticate in the.

That wasn't what I in the authenticate part, which I thought was here somewhere, handle authenticateasync here. It basically just asked the cookie, it's like, can you get me that cookie thing? And if There is a cookie. Can you go and get the. No, sorry, yeah, this actually does a bit more.

It returns an auth session for me straight away and from that auth session I can get stuff. But it basically allows me to store your authentication. So this one you can have a look at. It's a full end to end implementation of sign in, sign out and authentication and all of it. And if you do want to, if you get the repo of GitHub, the whole setup for the authentication facial recognition stuff is in there as well.

So you can run facial recognition on your local machine. Which was actually way better than I expected it to be. It's actually quite fast as well. Now I have another handler as well, so let's go in and try my other handler here. This thing here.

I'm gonna run out of time, but we'll figure it out. Random identity providers. So this is a remote version. So I'm gonna go and change this to random identity authentication. The random identity authentication defaults like that.

And we can log in using that. So we can see how that looks. Did I get everything? So this is a random identity provider. It's a remote thing.

It has a sign in scheme set to authentication scheme, a cookie authentication scheme. If we run this, I'm going to have to show you the network tab, which is very geeky. But that's the only way to prove that this actually works. So if we go here and we go to this thing here again, I'm now signed out, which is sweet. I don't have to sign out or anything, just restart the server and everybody gets locked out.

That's memory cache for the win, right? If we open this network tab, we go to network, we press sign in. You'll see here that I am now Darth Vader. I can also sign out and then sign in again. And then I'm Master Yoda.

So I built a random identity provider. You just get redirected to an identity provider and when it comes back you're someone. Why not? Very useful thing, right? Once again, I just want to show you remote authentication, but it is a bit more complicated if you go into the code.

If we have a look at the implementation, the random identity provider, if we look at the handler here it is. What happens in the handle challenge is that it basically does say, hey. It also has some events. I have this on redirect to identity provider event that you can plug into just like you have with the real remote one. Because this obviously doesn't implement.

Actually it does implement that, but I'VE overridden everything inside of it. And then it generates a correlation id, puts some state in there, redirects the user to the other end. When they come back, there's this handle, remote authenticateasync, which is looking at that callback path. And then inside here it does return. So I've built like a fake OpenID Connect workflow.

It passes a code back, and then I get the state, and then I look at the state to verify that the correlation stuff works. And then if the correlation stuff works, I basically send my code back to that other remote server and say, hey, I got this code thingamajig, can you please give me a user back? And then I get a principal back here. Then I have events for remote failure, events for principal received. And then finally I just return back and say, hey, everything went fine.

Here is my principal. You're logged in. So this is kind of what you would need to build if you want to build your own. And then for sign in and sign out, I've just overridden them to point them to sign in scheme in both cases. So if you try to use this thing to sign in, I will send you off to cookie authentication.

If you want to sign out, I redirect you to cookie authentication. If you want to get challenged, I will handle the challenge for you. So building one of those is not overly complicated. Let's see, I have another thing in here. We could go ahead.

I showed you this thing here. Add policy scheme. Now, this is another one of Chris's, you should not be coding kind of things. So if we take this thing here, I'm just gonna make some changes while I remember it. I'm gonna add both of these back in.

I'm gonna replace this here. I'm gonna turn this into default like that. So I've added this policy scheme called default, and that's gonna be my default authentication scheme. So it's gonna call the default scheme in all cases. And then when the user tries to get authenticated, it's going to end up in the policy scheme.

And the policy scheme uses this hey, forward selector thingy. And then it looks at the user agent string and says, hey, are you on an Edge browser? In that case, I'm going to use the random identity thing. And if you're on something else, I'm going to use the facial recognition one. No, not very useful.

But it was a nice way of showing you policies as well. Because the other one was, hey, it would be cool to have different domain names and things. Like this was Simple. I can show you how this works. So we'd expect Edge to be random identity provider.

Auth demo, sign out, sign in. That works. I'm now Mario now I'm ashamed of doing this. Don't in any way hold me to it, but I'm going to open Safari. It's the thing you never do, right?

Then you go there and you press sign in and I get the camera thing and I'm recognized and I have that logs in. So that's kind of. That's kind of neat. The policy scheme is actually quite useful and like I said, you can do a lot of stuff with it. This is dumb but multi tenancy if you get all of your tenants on different subdomains, for example or different domain names, then this can figure out which one to send them based on the subdomain, for example.

Or like some of the guys at the office, like the idea of if you're on local host then use another thing, right? I'm not a big fan of that because it still leaves that in your production code, but it kind of works. Now there's another thing you can do and that is claims transformation. How many of you have ever tried doing this? Add your own I claims transformation.

Anyone? Did you like it? No. I claims transformation is a very simple interface transformasync. You get a claims principle, you return the claims principle asynchronously.

Sweet. I can use that for a bunch of stuff. I've built one that basically puts a little claim in here that adds a arrival date, arrival date, claims transformation claim and it adds the time in UTC as a claim. And then because of the way this implemented, it also has to be idempotent because this thing can actually be called multiple times. Why?

Because reasons. So I built that because I want to show you something because I also inside of my random identity provider I did mention that I had events, right? So if we create one of these things here that listens to the on principle received event which is when the user comes back and the user has been authenticated and we run this. This is going to be fast. I hope.

Run this, it shows arrival date down here. Actually I'm going to sign out, I'm going to sign in again. Arrival date on principal received and arrival date claims transformation. They're pretty much identical. Not 100%.

But the scary thing that people don't know if it's like if I refresh this, the one that comes out of the claims translation is actually updated every time. This is not a thing that runs when you Sign in. This is a thing that runs on every incoming request and if you're unlucky, it runs on Every incoming CSS JavaScript image. Whatever request comes in, it will run this. So if you do things like, oh, I'm just going to save the user ID in a claim and then go and spring to run to the database to get the user's information on it when this happens.

Yeah, no, bad idea. Don't. Now I have a. My last demo. I got three minutes.

This is going to be probably one of those. Chris, you're stupid. I don't understand anything. That's okay. Download the code and look at it.

If you're running a bff, so you're running an API behind your front end and you use cookie authentication, which is what we should be doing, right? The problem is that when you run your API, like what I was doing here with making an API call and you're not authenticated and you use cookie, what happens? Well, if I do call API and you're not authenticated, I was logged in, I get a redirect. You do not want to get a redirect to a login page. When you do an API call, you want to get a 403 or a 401 back, right?

Probably 401. And you don't because the cookie authentication won't do that for you. So one area you can do is you can go into the cookie authentication, you can reconfigure on redirected login and you can do stuff in there. I obviously had to go further because stuff. So one thing you could do is if we go down to this thing here is that you could do add a group name to your API.

Once you have a group name which is a metadata thing that gets attached to your endpoint, you could then go in somewhere around, I think it's here I need to do it and go on redirect to identity provider and then I can look at the endpoint and I can look at the metadata and I can look at is there an endpoint group attribute? And if that endpoint group name is API, then I want to return a 401 and not a redirect. So I can basically reconfigure what it does. Really annoying having that in here. So I went, what can I do?

Well, chief over engineering officer at your service. You go and build an extension method that looks like this with API aware authentic. Then you go up here and you add this make API aware and you pass in cookie authentication defaults to say that I now want to make my cookie authentication defaults, like my cookie authentication thing aware of the fact that there are API endpoints that could be handled differently. Okay, that's kind of cool. Was that really all I need?

I think that's all I need. Let's see what happens if we run this. 401. Wait, what? Yeah, this is cool.

One minute. Fuck, this is not gonna work. I'm gonna show you quickly. So what this make API. So first of all, this thing here that says make with API aware auth all that really does is that it adds a metabol data at to the endpoint called IS API attributes or you can also do it using some other thing and then it adds an authentication scheme to it as a policy to make sure that there is a policy.

This thing up here? No, not that the make API aware. This thing here goes in and adds some authentication stuff. The thing that's interesting is the. I have my own custom authentication handler.

I have my authentic authentication handler provider decoration or options, blah, blah, blah. I. This thing here goes in. So that's the one I want. No, that's.

Sorry. Here I add my own I authentication handler provider. So I basically override the thing that creates authentication handlers in AS NET Core with my own implementation. That implementation implements basically allows you to go and find the handler. And if I have said make cookie authentication aware API aware, it has a list of schemes that should be aware of API stuff.

So whenever somebody tries to create a handler that is API aware, I go and create my handler and I create one of the other handlers and I take the original cookie handler and I put that inside of my handler, and then I return my handler and my handler, which is that specific handler that I had here. API aware authentication handler that implements sign in handler, which means every single API including I authentication request handler. And all it really does is that it forwards all of those requests. Authenticate and. Well, actually not forbid, but actually I didn't forward anything but authenticate, but you get the point.

The other ones, I basically go and say, okay, so you want to authenticate yourself. Oh, is this. Is this an API endpoint? Then I'm going to return 401 or 403. If not, I'm just going to pass it on to the thing that's behind it.

So basically I'm. I'm decorating the original handler with my own thing and then I have a handler instantiation thing that basically says I'm going to. You're asking for that handler. I'm actually going to take that handler and wrap it in my handler and give you my handler because it implements the same interface as yours. And then you end up with this.

Was this overly complicated to solve something that you can do with cookie authentication options? Yes. Was it fun? Yes. Sorry to say, but that's.

I have weird things that I like to do. Those were my demos. I'm two minutes over. I'm really sorry about that. Thank you so much for listening.

If you want to talk to me, I'll be around for a little while. You have my information there. The source code is on that thing. Just to fuck with you. Sorry the English, but you can't actually scan that because you will touch the other QR code as well, so it's unscannable.

But the address is bitly ancatd, which means asp.net core something. Anyway, thank you so much for listening.
