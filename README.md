# PhonepeAndroid
To integrate **PhonePe** as a payment option in your Android app, you need to follow the steps provided by PhonePe's **Payment Gateway API**. PhonePe offers a seamless payment experience via UPI, which is one of the fastest-growing payment methods in India.

Here’s a guide on how to integrate **PhonePe UPI** in your Android app.

### Steps to Integrate PhonePe UPI in Android

### 1. **Register for a PhonePe Merchant Account**
- Go to [PhonePe for Business](https://www.phonepe.com/business/) and sign up for a **merchant account**.
- Once you sign up, you will receive a **Merchant ID** (MID) and **Secret Key** that you will use to authenticate your transactions.

### 2. **PhonePe API Documentation**
- Read the [PhonePe API Documentation](https://developer.phonepe.com/docs/) to understand the various endpoints and the authentication mechanisms.
  
### 3. **Add Dependencies** (If PhonePe SDK is available)
If PhonePe provides an SDK, you should add the dependency in your `build.gradle` file (this would typically be mentioned in their developer documentation). If not, you will be dealing with **PhonePe Payment Gateway API** and HTTP calls for UPI payments.

### 4. **Generate Checksum for Secure Transactions**
To ensure transaction security, you need to generate a checksum using your **Merchant Key**. This checksum needs to be generated on your backend server.

Here is an example using **Node.js** to generate a checksum:

```javascript
const crypto = require('crypto');

const merchantId = 'yourMerchantID';
const merchantKey = 'yourSecretKey';
const amount = '10000';  // Transaction amount in paise (1 INR = 100 paise)
const transactionId = 'TXN123456789';  // Unique transaction ID
const redirectUrl = 'https://yourdomain.com/callback';

const data = merchantId + transactionId + amount + redirectUrl;
const checksum = crypto.createHmac('sha256', merchantKey).update(data).digest('hex');

console.log("Checksum:", checksum);
```

You will need to expose an API from your server that your Android app will call to get this checksum.

### 5. **Create the Payment Request in Your Android App**

You need to initiate a payment request from your Android app by opening PhonePe’s UPI interface. This is done using an **Intent**.

Here’s an example of how you can initiate a payment:

```kotlin
import android.content.Intent
import android.net.Uri
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.Toast

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Replace with your actual parameters
        val amount = "10000" // Amount in paise
        val transactionId = "TXN123456789"
        val merchantId = "yourMerchantID"
        val checksum = "checksumGeneratedFromServer"

        // Intent for UPI Payment through PhonePe
        val uri = Uri.Builder()
            .scheme("upi")
            .authority("pay")
            .appendQueryParameter("pa", "your@phonepe") // UPI ID
            .appendQueryParameter("pn", "MerchantName") // Merchant Name
            .appendQueryParameter("mc", "") // Merchant Code (optional)
            .appendQueryParameter("tid", transactionId) // Transaction ID
            .appendQueryParameter("tr", transactionId) // Transaction Reference ID
            .appendQueryParameter("tn", "Description") // Description
            .appendQueryParameter("am", "100.00") // Amount
            .appendQueryParameter("cu", "INR") // Currency
            .appendQueryParameter("url", "https://yourcallbackurl.com") // Callback URL
            .appendQueryParameter("crc", checksum) // Checksum
            .build()

        val upiIntent = Intent(Intent.ACTION_VIEW, uri)
        upiIntent.setPackage("com.phonepe.app") // Specific to PhonePe UPI

        // Start the PhonePe payment activity
        try {
            startActivityForResult(upiIntent, 1001)
        } catch (e: Exception) {
            Toast.makeText(this, "PhonePe not installed.", Toast.LENGTH_LONG).show()
        }
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (requestCode == 1001) {
            if (data != null) {
                val response = data.getStringExtra("response")
                Toast.makeText(this, "Transaction Response: $response", Toast.LENGTH_LONG).show()
                // Handle the transaction response
            } else {
                Toast.makeText(this, "Transaction canceled.", Toast.LENGTH_LONG).show()
            }
        }
    }
}
```

### 6. **Handle the Transaction Response**
When the transaction completes, the result will be passed back in the `onActivityResult` method of your activity. You need to handle this result and verify the transaction status by calling your server API or PhonePe’s API.

- The possible statuses are **SUCCESS**, **FAILURE**, or **PENDING**.
- Based on the status, update the order status on your server and notify the user.

### 7. **Verify Payment on the Backend**
After receiving the payment response, always verify the transaction by calling PhonePe’s backend API. This ensures the payment was actually successful.

```kotlin
// Example of a verification API call to your backend
fun verifyPayment(transactionId: String) {
    // Make a server call to verify the transaction
}
```

### 8. **Test the Integration**
- Use **sandbox credentials** for testing the integration.
- Ensure that everything works correctly in your testing environment before switching to production.

### 9. **Go Live**
Once your integration is working correctly in the test environment, you can go live by switching to the **production** environment in your backend and app.

### Conclusion:
- **PhonePe UPI** integration provides a fast and secure way for users to make payments directly from your app.
- You need to handle transaction responses properly and verify the transaction status with your backend to ensure a seamless experience.
