# newline-rust-bug
Bug on Page 83, creating useful errors: Appstate cannot be extracted inside Error handling function

# What is the issue ? 
On Page 83 it is explained how to add an error handling function for the case that the post data cannot be serialized into the `PostInput` type.
According to the book, the `AppState` is supposed to be retrievable as seen in src/lib.rs:89 - 90:  
```
let extns = req.extensions();
let state = extns.get::<web::Data<AppState>>().unwrap();
```

However, this unwrap() fails, leaving you with an `unhandled None` value.

Besides this, according to the crate´s documentation, the AppState is supposed to be to retrieved like this: https://docs.rs/actix-web/1.0.0/actix_web/struct.HttpRequest.html#method.app_data which also leads to the same error, unfortunately.

Another person, seemingly also following the book, had the same issue as me: https://github.com/actix/actix-web/issues/1628
As can be seen, in a more recent version of the library its API is slightly adjusted (the book uses 1.0, the current one is 4.0.x), and the issue seems to have disappeared.

The bug can be reproduced as follows:

1. Clone the repository
2. `cargo run` the application
3. Make sure `POST`ing correct data works:
```
curl --location --request POST 'localhost:8080/send' \
--header 'Content-Type: application/json' \
--data-raw '{
    "message": "test"
}'
```
4. Cause the bug to happen by altering the JSON data:
```
curl --location --request POST 'localhost:8080/send' \
--header 'Content-Type: application/json' \
--data-raw '{
    "msg": "test"
}'
```
5. see console: 
```
thread 'actix-rt:worker:1' panicked at 'called `Option::unwrap()` on a `None` value', src/lib.rs:90:52
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
Panic in Arbiter thread.
```
