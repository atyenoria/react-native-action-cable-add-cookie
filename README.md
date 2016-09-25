# Setup

- change line in Libraries/RCTWebsocket.xcodeproj/RCTSRWebSocket.m
```
- (void)didConnect
{
  RCTSRLog(@"Connected");
  CFHTTPMessageRef request = CFHTTPMessageCreateRequest(NULL, CFSTR("GET"), (__bridge CFURLRef)_url, kCFHTTPVersion1_1);

  // Set host first so it defaults
  CFHTTPMessageSetHeaderFieldValue(request, CFSTR("Host"), (__bridge CFStringRef)(_url.port ? [NSString stringWithFormat:@"%@:%@", _url.host, _url.port] : _url.host));

  NSMutableData *keyBytes = [[NSMutableData alloc] initWithLength:16];
  SecRandomCopyBytes(kSecRandomDefault, keyBytes.length, keyBytes.mutableBytes);
  _secKey = [keyBytes base64EncodedStringWithOptions:0];
  assert([_secKey length] == 24);

  CFHTTPMessageSetHeaderFieldValue(request, CFSTR("Upgrade"), CFSTR("websocket"));
  CFHTTPMessageSetHeaderFieldValue(request, CFSTR("Cookie"), (__bridge CFStringRef)_requestedProtocols[2]);
  CFHTTPMessageSetHeaderFieldValue(request, CFSTR("Connection"), CFSTR("Upgrade"));
  CFHTTPMessageSetHeaderFieldValue(request, CFSTR("Sec-WebSocket-Key"), (__bridge CFStringRef)_secKey);
  CFHTTPMessageSetHeaderFieldValue(request, CFSTR("Sec-WebSocket-Version"), (__bridge CFStringRef)[NSString stringWithFormat:@"%ld", (long)_webSocketVersion]);

  CFHTTPMessageSetHeaderFieldValue(request, CFSTR("Origin"), (__bridge CFStringRef)_url.RCTSR_origin);

  if (_requestedProtocols) {
    CFHTTPMessageSetHeaderFieldValue(request, CFSTR("Sec-WebSocket-Protocol"), (__bridge CFStringRef)[_requestedProtocols componentsJoinedByString:@", "]);
  }

  [_urlRequest.allHTTPHeaderFields enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
    CFHTTPMessageSetHeaderFieldValue(request, (__bridge CFStringRef)key, (__bridge CFStringRef)obj);
  }];

  NSData *message = CFBridgingRelease(CFHTTPMessageCopySerializedMessage(request));

  CFRelease(request);

  [self _writeData:message];
  [self _readHTTPHeader];
}
```


- set cookie like below

```
    this.App = {};
    this.App.cable = actioncable-add-cookie.createConsumer("wss://localhost:4000/cable","Your COOKIE")
    this.App.room_chat = this.App.cable.subscriptions.create("RoomChannel", {
      connected: () => {
        console.log("connected: action cable")
      },
      disconnected: () => {
        console.log("disconnected: action cable")
      },
      received: (data) => {
      }
    });
})
```

Comp
