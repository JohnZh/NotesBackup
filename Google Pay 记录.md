# Google Pay 4 步流程

支付流程 4 步：

1. 点击 Google Pay UI，然后出现一个清单。出现一个清单列表。列表上有支持的支付方式（信用卡，PayPal）
2. 用户选择一个支付方式，然后GP会根据选择的支付方式安全地返回一个 Token
3. 用户提交支付的 Token。伴随提交交易详情到我们自己的后台（BackEnd）
4. BE 会处理这次交易，并发送Token 到这个支付供应商（比如信用卡，通知银行扣多少钱）



# 6 个启动步骤

1. Review 和遵守服务协议和政策（[Terms of Service](https://payments.developers.google.com/terms/sellertos) and [Acceptable Use Policy](https://payments.developers.google.com/terms/aup)）
2. 找到你要使用的支付方式的参考文档（[supported processors and gateways list](https://developers.google.com/pay/api#participating-processors)）
3. Review 和遵守 GP 的品牌指导方针（[Brand guidelines](https://developers.google.com/pay/api/android/guides/brand-guidelines)）
4. 完成指导和集成清单（[Tutorial](https://developers.google.com/pay/api/android/guides/tutorial) and [Integration checklist](https://developers.google.com/pay/api/android/guides/test-and-deploy/integration-checklist)）
5. 使用Google Pay 和 Wallet 控制后台（[Google Pay and Wallet Console](https://pay.google.com/business/console)）控制产品访问 Google Pay API（[Request production access](https://developers.google.com/pay/api/android/guides/test-and-deploy/request-prod-access) ）
6. 确保设备有 Google Play 服务具备 18.0.0 或更高版本



# Demo 运行问题

会提示两个错误：

1. Could not resolve com.android.tools.build:gradle:8.0.0. 相关

   fix：修改 gradle 的 jdk 版本为内置 java11

2. The project is using an incompatible version (AGP 8.0.0) of the Android Gradle plugin. Latest supported version is AGP 7.4.2

   ```kotlin
   plugins {
   		// 改动 8.0.0 -> 7.4.2
       id 'com.android.application' version '7.4.2' apply false
       id 'org.jetbrains.kotlin.android' version '1.8.10' apply false
   }
   ```



# 9 步集成 Google Pay

## S1: 定义 Google Pay API  版本

作为一个基础基础请求体，后续会应用到其他的请求里，作为 base

```kotlin
// PaymentsUtil.kt
private val baseRequest = JSONObject().apply {
  put("apiVersion", 2)
  put("apiVersionMinor", 0)
}
```

疑问：这个版本后期是否要更改？



## S6: 确定 Google Pay API 是否可用

用来更新 Google Pay 的 UI 是否可用，这里需要先初始化 `paymentsClient`，用来发生请求的

```kotlin
// PaymentsUtil.kt
fun isReadyToPayRequest(): JSONObject? {
  return try {
    baseRequest.apply {
      put("allowedPaymentMethods", JSONArray().put(baseCardPaymentMethod()))
    }

  } catch (e: JSONException) {
    null
  }
}

private fun fetchCanUseGooglePay() {
  val isReadyToPayJson = PaymentsUtil.isReadyToPayRequest()
  if (isReadyToPayJson == null) _canUseGooglePay.value = false

  val request = IsReadyToPayRequest.fromJson(isReadyToPayJson.toString())
  val task = paymentsClient.isReadyToPay(request)
  task.addOnCompleteListener { completedTask ->
    try {
      _canUseGooglePay.value = completedTask.getResult(ApiException::class.java)
    } catch (exception: ApiException) {
      Log.w("isReadyToPay failed", exception)
      _canUseGooglePay.value = false
    }
  }
}
```



## S2-S3: 请求支付（不含 Google Pay Button）

假定可以不使用 Google Pay Button，点击自己的按钮进行支付的请求

```kotlin
// The price provided to the API should include taxes and shipping.
// This price is not displayed to the user.
val dummyPriceCents = 100L
val shippingCostCents = 900L
val task = model.getLoadPaymentDataTask(dummyPriceCents + shippingCostCents)

fun getLoadPaymentDataTask(priceCents: Long): Task<PaymentData> {
  val paymentDataRequestJson = PaymentsUtil.getPaymentDataRequest(priceCents)
  val request = PaymentDataRequest.fromJson(paymentDataRequestJson.toString())
  return paymentsClient.loadPaymentData(request)
}

fun getPaymentDataRequest(priceCemts: Long): JSONObject {
  return baseRequest.apply {
    put("allowedPaymentMethods", JSONArray().put(cardPaymentMethod())) // 包含了请求支付提供商 Token 的配置
    put("transactionInfo", getTransactionInfo(priceCemts.centsToString()))
    put("merchantInfo", merchantInfo)

    // An optional shipping address requirement is a top-level property of the
    // PaymentDataRequest JSON object.
    val shippingAddressParameters = JSONObject().apply {
      put("phoneNumberRequired", false)
      put("allowedCountryCodes", JSONArray(listOf("US", "GB")))
    }
    put("shippingAddressParameters", shippingAddressParameters)
    put("shippingAddressRequired", true)
  }
}
```

### S2: （重要）为支付提供商请求支付令牌

```kotlin
private fun cardPaymentMethod(): JSONObject {
  val cardPaymentMethod = baseCardPaymentMethod()
  cardPaymentMethod.put("tokenizationSpecification", gatewayTokenizationSpecification())

  return cardPaymentMethod
}

/**
     * Gateway Integration: Identify your gateway and your app's gateway merchant identifier.
     *
     *
     * The Google Pay API response will return an encrypted payment method capable of being charged
     * by a supported gateway after payer authorization.
     *
     *
     * TODO: Check with your gateway on the parameters to pass and modify them in Constants.java.
     *
     * @return Payment data tokenization for the CARD payment method.
     * @throws JSONException
     * See [PaymentMethodTokenizationSpecification](https://developers.google.com/pay/api/android/reference/object.PaymentMethodTokenizationSpecification)
     */
private fun gatewayTokenizationSpecification(): JSONObject {
  return JSONObject().apply {
    put("type", "PAYMENT_GATEWAY")
    put("parameters", JSONObject(Constants.PAYMENT_GATEWAY_TOKENIZATION_PARAMETERS))
  }
}

/**
     * Custom parameters required by the processor/gateway.
     * In many cases, your processor / gateway will only require a gatewayMerchantId.
     * Please refer to your processor's documentation for more information. The number of parameters
     * required and their names vary depending on the processor.
     *
     * @value #PAYMENT_GATEWAY_TOKENIZATION_PARAMETERS
     */
val PAYMENT_GATEWAY_TOKENIZATION_PARAMETERS = mapOf(
  "gateway" to PAYMENT_GATEWAY_TOKENIZATION_NAME,
  "gatewayMerchantId" to "exampleGatewayMerchantId"
)

private const val PAYMENT_GATEWAY_TOKENIZATION_NAME = "example"
```

`example` 和 `exampleGatewayMerchantId` 是需要根据文档进行替换成正确的值的

注意：测试环境下 `example` 这个值是合法的 gateway 名称



### S3: 定义支持的支付卡片的组织

这个定义在 `baseCardPaymentMethod()`

```kotlin
private fun cardPaymentMethod(): JSONObject {
  val cardPaymentMethod = baseCardPaymentMethod()
  cardPaymentMethod.put("tokenizationSpecification", gatewayTokenizationSpecification())

  return cardPaymentMethod
}

private fun baseCardPaymentMethod(): JSONObject {
  return JSONObject().apply {

    val parameters = JSONObject().apply {
      put("allowedAuthMethods", allowedCardAuthMethods)
      put("allowedCardNetworks", allowedCardNetworks) // 支付支付卡的组织，eg. VISA
      put("billingAddressRequired", true)
      put("billingAddressParameters", JSONObject().apply {
        put("format", "FULL")
      })
    }

    put("type", "CARD")
    put("parameters", parameters)
  }
}

private val allowedCardNetworks = JSONArray(Constants.SUPPORTED_NETWORKS)

val SUPPORTED_NETWORKS = listOf(
  "AMEX",
  "DISCOVER",
  "JCB",
  "MASTERCARD",
  "VISA")
```

返回的卡片根据 Google 认证模式也可以分为两种：因此你也可以配置返回那种认证的卡片

```kotlin
private val allowedCardAuthMethods = JSONArray(listOf(
  "PAN_ONLY",
  "CRYPTOGRAM_3DS"))
```



## S4: 描述你允许的支付方式

描述你的 app 支持 CARD 的支付方式，结合支持的认证方法和卡片金融组织

```kotlin
private fun baseCardPaymentMethod(): JSONObject {
  return JSONObject().apply {

    val parameters = JSONObject().apply {
      put("allowedAuthMethods", allowedCardAuthMethods)
      put("allowedCardNetworks", allowedCardNetworks)
      put("billingAddressRequired", true)
      put("billingAddressParameters", JSONObject().apply {
        put("format", "FULL")
      })
    }

    put("type", "CARD") // 支持 CARD
    put("parameters", parameters)
  }
}
```



## S5:  初始化 S6 中使用的 PaymentsClient

```kotlin
fun createPaymentsClient(activity: Activity): PaymentsClient {
  val walletOptions = Wallet.WalletOptions.Builder()
  .setEnvironment(Constants.PAYMENTS_ENVIRONMENT)
  .build()

  return Wallet.getPaymentsClient(activity, walletOptions)
}

const val PAYMENTS_ENVIRONMENT = WalletConstants.ENVIRONMENT_TEST
```



## S6 即 S6



## S7: 创建 PaymentDataRequest 对象（见 S2-S3）

```kotlin
// https://developers.google.com/pay/api/android/reference/request-objects#PaymentDataRequest
val task = model.getLoadPaymentDataTask(dummyPriceCents + shippingCostCents)

fun getLoadPaymentDataTask(priceCents: Long): Task<PaymentData> {
  val paymentDataRequestJson = PaymentsUtil.getPaymentDataRequest(priceCents)
  val request = PaymentDataRequest.fromJson(paymentDataRequestJson.toString())
  return paymentsClient.loadPaymentData(request)
}

fun getPaymentDataRequest(priceCemts: Long): JSONObject {
  return baseRequest.apply {
    put("allowedPaymentMethods", JSONArray().put(cardPaymentMethod()))
    put("transactionInfo", getTransactionInfo(priceCemts.centsToString())) // 交易的信息
    put("merchantInfo", merchantInfo)

    // An optional shipping address requirement is a top-level property of the
    // PaymentDataRequest JSON object.
    val shippingAddressParameters = JSONObject().apply {
      put("phoneNumberRequired", false)
      put("allowedCountryCodes", JSONArray(listOf("US", "GB")))
    }
    put("shippingAddressParameters", shippingAddressParameters)
    put("shippingAddressRequired", true)
  }
}

// 交易的信息 transaction price and the status of the provided price
// https://developers.google.com/pay/api/android/reference/request-objects#TransactionInfo
private fun getTransactionInfo(price: String): JSONObject {
  return JSONObject().apply {
    put("totalPrice", price)
    put("totalPriceStatus", "FINAL")
    put("countryCode", Constants.COUNTRY_CODE)
    put("currencyCode", Constants.CURRENCY_CODE)
  }
}
```

### 提供用户可视化的商家名称

```kotlin
private val merchantInfo: JSONObject =
            JSONObject().put("merchantName", "Example Merchant")
```



## S8: 注册支付事件处理者

添加一个点击发送数据事件

```kotlin
googlePayButton.setOnClickListener { requestPayment() }

private fun requestPayment() {

  // Disables the button to prevent multiple clicks.
  googlePayButton.isClickable = false

  // The price provided to the API should include taxes and shipping.
  // This price is not displayed to the user.
  val dummyPriceCents = 100L
  val shippingCostCents = 900L
  val task = model.getLoadPaymentDataTask(dummyPriceCents + shippingCostCents)

  task.addOnCompleteListener { completedTask ->
    if (completedTask.isSuccessful) {
      completedTask.result.let(::handlePaymentSuccess)
    } else {
      when (val exception = completedTask.exception) {
        is ResolvableApiException -> {
          resolvePaymentForResult.launch(
            IntentSenderRequest.Builder(exception.resolution).build()
          )
        }

        is ApiException -> {
          handleError(exception.statusCode, exception.message)
        }

        else -> {
          handleError(
            CommonStatusCodes.INTERNAL_ERROR, "Unexpected non API" +
            " exception when trying to deliver the task result to an activity!"
          )
        }
      }
    }

    // Re-enables the Google Pay payment button.
    googlePayButton.isClickable = true
  }
}
```

这里也可以使用 [`AutoResolveHelper`](https://developers.google.com/android/reference/com/google/android/gms/wallet/AutoResolveHelper) 来自动解析 Task，把结果通过 `Activity#onActivityResult` 返回



## S9: 处理支付的 Response

```kotlin
public override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
  when (requestCode) {
    // Value passed in AutoResolveHelper
    LOAD_PAYMENT_DATA_REQUEST_CODE -> {
      when (resultCode) {
        RESULT_OK ->
        data?.let { intent ->
                   PaymentData.getFromIntent(intent)?.let(::handlePaymentSuccess)
                  }

        RESULT_CANCELED -> {
          // The user cancelled the payment attempt
        }

        AutoResolveHelper.RESULT_ERROR -> {
          AutoResolveHelper.getStatusFromIntent(data)?.let {
            handleError(it.statusCode)
          }
        }
      }

      // Re-enables the Google Pay payment button.
      googlePayButton.isClickable = true
    }
  }
}

private fun handlePaymentSuccess(paymentData: PaymentData) {
  val paymentInformation = paymentData.toJson()

  try {
    // Token will be null if PaymentDataRequest was not constructed using fromJson(String).
    val paymentMethodData =
    JSONObject(paymentInformation).getJSONObject("paymentMethodData")
    val billingName = paymentMethodData.getJSONObject("info")
    .getJSONObject("billingAddress").getString("name")
    Log.d("BillingName", billingName)

    Toast.makeText(
      this,
      getString(R.string.payments_show_name, billingName),
      Toast.LENGTH_LONG
    ).show()

    // Logging token string.
    Log.d(
      "Google Pay token", paymentMethodData
      .getJSONObject("tokenizationData")
      .getString("token")
    )

    startActivity(Intent(this, CheckoutSuccessActivity::class.java))

  } catch (error: JSONException) {
    Log.e("handlePaymentSuccess", "Error: $error")
  }
}
```

这些支付信息后续需要和后端一起处理
