AFNetworking-Synchronous
========================

A minimal category which extends [AFNetworking][] to support synchronous
requests.

## Example (2.x)

```rb
pod 'AFNetworking', '~> 2.0'
pod 'AFNetworking-Synchronous/2.x'
```

```objective-c
#import <AFNetworking.h>
#import <AFHTTPRequestOperationManager+Synchronous.h>

AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
NSError *error = nil;
NSData *result = [manager syncGET:@"/document/123"
parameters:paramDict
operation:NULL
error:&error];
```

## Example (3.x)

```rb
pod 'AFNetworking', '~> 2.0'
pod 'AFNetworking-Synchronous/2.x'
```

```objective-c
#import <AFNetworking.h>
#import <AFHTTPSessionManager+Synchronous.h>

AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
NSError *error = nil;
NSData *result = [manager GETSynchronously:@"/document/123"
parameters:paramDict
progress:nil
operation:NULL
error:&error];
```

## Discussion

### First, consider adopting an asynchronous design

Before you decide to use this category, consider whether you can adopt an asynchronous design instead. As @mattt wrote, asynchronism a tough thing to
get your head around, but it's well worth the mental overhead. Rather than creating methods that fetch and return network data, use blocks or delegate methods to call back with the results when you have them.

Using the asynchronous API has many advantages:

- When you start an operation on the main thread, you return control to the run loop immediately, so your UI can remains responsive. Blocking the main thread for a long time is never a good idea. "Be responsive," Appe urges in the OS X user experience guidelines. Asynchronous network operations allow
you to do that.
- AFNetworking makes asynchronous code easy to write and easy to read. With block-based success and failure handlers, you don't need to implement
delegate protocols or provide selectors for callbacks.
- AFNetworking and Grand Central Dispatch take care of threading for you, so your code does not need to manage threads, run selectors in the background,
or invoke dispatch_async. Your completion blocks will be executed on the main thread (unless you configure the operations otherwise).
- You can provide a better user experience while waiting for a response. Networks are unreliable, particularly for mobile users, and servers can be bogged down. Your users' experiences will be better if you design for a slow connection, which you can only do asynchronously.

However, in some cases, a synchronous response is better, such as when the document architecture or another framework is handling the multithreading for you, and expects a synchronous result. This code attempts to provide a safe and reliable way to use the framework synchronously.

While it overrides the default success and failure queues to avoid a deadlock, it can't anticipate every possible situation. In particular, you should not set the queue from which you're invoking as the processing queue, which will cause a deadlock.

### The Main Thread

You shouldn't call these methods from the main thread. On iOS, if your application enters the background while one of these methods is running on the main thread, a deadlock may result and your application could be terminated.

[AFNetworking]: https://github.com/AFNetworking/AFNetworking
