## Integration

- [Integration](#integration)
  - [Set up PostFinanceCheckout](#set-up-PostFinanceCheckout)
  - [Create transaction](#create-transaction)
  - [Collect payment details](#collect-payment-details)
  - [Handle result](#handle-result)
  - [Verify payment](#verify-payment)

### Set up PostFinanceCheckout

To use the Android Payment SDK, you need a [PostFinanceCheckout account](https://checkout.postfinance.ch/user/signup). After signing up, set up your space and enable the payment methods you would like to support.

### Create transaction

For security reasons, your app cannot create transactions and fetch access tokens. This has to be done on your server by talking to the [PostFinanceCheckout Web Service API](https://checkout.postfinance.ch/en-us/doc/api/web-service). You can use one of the official SDK libraries to make these calls.

To use the Android Payment SDK to collect payments, an endpoint needs to be added on your server that creates a transaction by calling the [create transaction](https://checkout.postfinance.ch/doc/api/web-service#transaction-service--create) API endpoint. A transaction holds information about the customer and the line items and tracks charge attempts and the payment state.

Once the transaction has been created, your endpoint can fetch an access token by calling the [create transaction credentials](https://checkout.postfinance.ch/doc/api/web-service#transaction-service--create-transaction-credentials) API endpoint. The access token is returned and passed to the Android Payment SDK.

```bash
# Create a transaction
curl 'https://checkout.postfinance.ch/api/transaction/create?spaceId=1' \
  -X "POST" \
  -d "{{TRANSACTION_DATA}}"

# Fetch an access token for the created transaction
curl 'https://checkout.postfinance.ch/api/transaction/createTransactionCredentials?spaceId={{SPACE_ID}}&id={{TRANSACTION_ID}}' \
  -X 'POST'
```

### Collect payment details

Before launching the Android Payment SDK to collect the payment, your checkout page should show the total amount, the products that are being purchased and a checkout button to start the payment process.

Is recommended to initialize `PostFinanceCheckoutSdk` in `Application` class. You can always access `PostFinanceCheckoutSdk` instance and `paymentResult` from everywhere in your app.

```kotlin
// ...
import android.app.Application
import ch.postfinance.PostFinanceCheckoutSdk
import ch.postfinance.event.OnResultEventListener
import ch.postfinance.event.PaymentResult

class MyApp : Application() {

    private val _mainState = MutableLiveData<PaymentResult>()
    val mainState: LiveData<PaymentResult> get() = _mainState

    override fun onCreate() {
        super.onCreate()
        PostFinanceCheckoutSdk.init(listener = object : OnResultEventListener{
            override fun paymentResult(paymentResult: PaymentResult) {
                _mainState.postValue(paymentResult)
            }
        })
    }
    //...
}
```

When the customer taps the checkout button, call your endpoint that creates the transaction and returns the access token, and launch the payment dialog.

```kotlin
// ...

class MainActivity : AppCompatActivity() {
    lateinit var btnCheckout: Button

    var token = "" // to get token you have to create transaction. Checkout previous section #Create transaction

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        btnCheckout.setOnClickListener {
            PostFinanceCheckoutSdk.instance?.launch(token, this) ?: run {
                Log.e(
                    "Mobile SDK",
                    "SDK is not initialized. Did you forget to run init on Application?"
                )
            }
        }

    }

    // ...
}
```

After the customer completes the payment, the dialog dismisses and the `paymentResult` method is called.

### Handle result

The response object contains these properties:

- `code` describing the result's type.

  | Code | Description |
  | --- | --- |
  | `COMPLETED` | The payment was successful. |
  | `FAILED` | The payment failed. Check the `message` for more information. |
  | `CANCELED` | The customer canceled the payment. |
  | `PENDING` | The customer has aborted the payment process, so the payment is in a temporarily pending state. It will eventually reach a final status (successful or failed), but it may take a while. Wait for a webhook notification and use the PostFinanceCheckout API to retrieve the status of the transaction and inform the customer that the payment is pending. |
  | `TIMEOUT` | Token for this transaction expired. App will be closed and third-party app will get this message. For opening payment sdk third party app have to refetch token |

- `message` providing a localized error message that can be shown to the customer.

```kotlin

class MainActivity : AppCompatActivity() {
    // ...
    lateinit var txtResultMessage: TextView


    override fun onCreate(savedInstanceState: Bundle?) {
      //...
      txtResultMessage = findViewById(R.id.txtResultMessage)

      init()
    }

    private fun init() {
        (application as? MyApp)?.mainState?.observe(this) { paymentResult ->

            txtResultMessage.text = paymentResult.code.toString()
            val colorCodeMap = mapOf(
                PaymentResultEnum.FAILED to Color.RED,
                PaymentResultEnum.COMPLETED to Color.GREEN,
                PaymentResultEnum.CANCELED to Color.parseColor("#ffa500")
            )
            colorCodeMap[paymentResult.code]?.let { it1 -> txtResultMessage.setTextColor(it1) }
        }
    }

    // ...
}


```

### Verify payment

As customers could quit the app or lose network connection before the result is handled or malicious clients could manipulate the response, it is strongly recommended to set up your server to listen for webhook events the get transactions' actual states. Find more information in the [webhook documentation](https://checkout.postfinance.ch/en-us/doc/webhooks).
