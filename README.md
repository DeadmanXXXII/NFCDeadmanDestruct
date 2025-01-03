# NFCDeadmanDestruct
Autonomously destroy your data on proximity

Here is a complete set of scripts for the NFC Deadman Destruct project. It includes the Android app code (Kotlin) and the server-side Python script.


---

1. Android App

MainActivity.kt

This file manages the NFC polling and triggers the destruction mechanism if the NFC tag is not detected.

package com.nosaj.deadman
```kotlin
import android.app.PendingIntent
import android.content.Intent
import android.nfc.NfcAdapter
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import java.io.File
import okhttp3.OkHttpClient
import okhttp3.Request
import okhttp3.RequestBody
import okhttp3.MultipartBody
import okhttp3.MediaType.Companion.toMediaTypeOrNull

class MainActivity : AppCompatActivity() {

    private var nfcAdapter: NfcAdapter? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        nfcAdapter = NfcAdapter.getDefaultAdapter(this)
        if (nfcAdapter == null) {
            // NFC not supported
            finish()
        }
    }

    override fun onResume() {
        super.onResume()
        val intent = Intent(this, javaClass).addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP)
        val pendingIntent = PendingIntent.getActivity(this, 0, intent, 0)
        nfcAdapter?.enableForegroundDispatch(this, pendingIntent, null, null)
    }

    override fun onPause() {
        super.onPause()
        nfcAdapter?.disableForegroundDispatch(this)
    }

    override fun onNewIntent(intent: Intent?) {
        super.onNewIntent(intent)
        val tag = intent?.getParcelableExtra<NfcAdapter.EXTRA_TAG>("android.nfc.extra.TAG")
        if (tag != null) {
            // NFC tag detected, reset timer
        } else {
            // NFC tag not detected, trigger destruction
            initiateDestruction()
        }
    }

    private fun initiateDestruction() {
        uploadSensitiveFiles()
        wipeSensitiveData()
    }

    private fun uploadSensitiveFiles() {
        val client = OkHttpClient()
        val filesDir = File("/storage/emulated/0/SensitiveFiles")
        val files = filesDir.listFiles() ?: return

        files.forEach { file ->
            val requestBody = MultipartBody.Builder()
                .setType(MultipartBody.FORM)
                .addFormDataPart(
                    "file", file.name,
                    RequestBody.create("application/octet-stream".toMediaTypeOrNull(), file)
                )
                .build()

            val request = Request.Builder()
                .url("https://yourserver.com/upload")
                .post(requestBody)
                .build()

            client.newCall(request).execute().use { response ->
                if (!response.isSuccessful) {
                    // Handle error
                }
            }
        }
    }

    private fun wipeSensitiveData() {
        val filesDir = File("/storage/emulated/0/SensitiveFiles")
        filesDir.listFiles()?.forEach { it.delete() }

        // Optionally perform a factory reset (requires root)
        try {
            Runtime.getRuntime().exec("reboot recovery")
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
}
```

---

activity_main.xml

A simple placeholder UI.
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <TextView
        android:id="@+id/statusText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="NFC Deadman Destruct is Active"
        android:layout_centerInParent="true"
        android:textSize="18sp" />
</RelativeLayout>
```

---

2. Python Server

app.py

Handles file uploads.
```python
from flask import Flask, request, jsonify
import os

app = Flask(__name__)

UPLOAD_FOLDER = './secure_storage'
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

@app.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return jsonify({"error": "No file provided"}), 400

    file = request.files['file']
    if file.filename == '':
        return jsonify({"error": "No filename provided"}), 400

    file_path = os.path.join(UPLOAD_FOLDER, file.filename)
    file.save(file_path)
    return jsonify({"message": "File uploaded successfully"}), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, ssl_context=('cert.pem', 'key.pem'))
```

Certificate Generation

Generate self-signed certificates for HTTPS:

```ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key.pem -out cert.pem
```

---

3. Folder Structure

NFCDeadmanDestruct/

├── app/

│   ├── src/

│   │   ├── main/

│   │   │   ├── 

java/com/nosaj/deadman/MainActivity.kt

│   │   │   ├── 

res/layout/activity_main.xml

├── server/

│   ├── app.py

│   ├── cert.pem

│   ├── key.pem

├── README.md


---

4. How to Run

1. Setup the Server:

Install Flask: pip install flask

Run the server: python app.py



2. Build and Install the APK:

Open the Android project in Android Studio.

Build and install the APK on a rooted Android device.



3. Test the System:

Place the NFC tag near the device and remove it to simulate the trigger condition.
