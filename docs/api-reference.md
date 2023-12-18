## API reference

| API | Type | Description |
| --- | :-: | --- |
| `PostFinanceCheckoutSdk.init(listener: OnResultEventListener)` | constructor | Initialization of SDK. Both Parameters are required! |
| `OnResultEventListener` | interface | Interface for handling post-payment events `paymentResult` |
| `fun paymentResult(paymentResult: PaymentResult)` | function | Result handler for transaction state |
| `PostFinanceCheckoutSdk.instance?.launch(token: String)` | function | Opening payment dialog (activity) |
| `PostFinanceCheckoutSdk.instance?.setDarkTheme(theme: JSONObject)` | function | Can override the whole dark theme or just some specific color. All colors are in json format |
| `PostFinanceCheckoutSdk.instance?.setLightTheme(theme: JSONObject)` | function | Can override the whole light theme or just some specific color. All colors are in json format |
| `PostFinanceCheckoutSdk.instance?.setCustomTheme(theme: JSONObject?, baseTheme: ThemeEnum)` | function | Force to use only this theme (independent on user's setup). Can override default light/dark theme and force to use it or completely replace all or specific colors |
| `PostFinanceCheckoutSdk.instance?.setAnimation(type: AnimationEnum)` | function | Defining type of animation for moving between the pages |
