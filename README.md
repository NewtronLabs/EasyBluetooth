# Easy Bluetooth

The EasyBluetooth library allows the fast creation of Bluetooth connections between devices. 

----


## How to Use 

### Setup

Include the below dependencies in your `build.gradle` project.

```gradle
buildscript {
    repositories {
        jcenter()
        maven { url "http://code.newtronlabs.com:8081/artifactory/libs-release-local" }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.3'
        classpath "com.newtronlabs.android:plugin:1.1.0"
    }
}

allprojects {
    repositories {
        jcenter()
        maven { url "http://code.newtronlabs.com:8081/artifactory/libs-release-local" }
    }
}
```

In the `build.gradle` for your app.

```gradle
apply plugin: 'com.newtronlabs.android'

dependencies {
    provided 'com.newtronlabs.easybluetooth:easybluetooth:2.0.0'
}
```


### EasyBluetooth - Server
In order to create a server we build a Bluetooth Server and accept clients.

```java
// Define your service and it's UUID
IBluetoothServer btServer = new BluetoothServer.Builder(this.getApplicationContext(),
"EasyBtService",ParcelUuid.fromString("00001101-0000-1000-8000-00805F9B34FB"))
.build();

if(btServer == null)
{
    // Server could not be created.
}
else
{
    // Block until a client connects.
    IBluetoothClient btClient = btServer.accept();
    // Set a data callback to receive data from the remote device.
    btClient.setDataCallback(new SampleDataCallback());
    // Set a connection callback to be notified of connection changes.
    btClient.setConnectionCallback(new SampleConnectionCallback());
    // Set a data send callback to be notified when data is sent of fails to send.
    btClient.setDataSentCallback(new SampleDataSentCallback());
    btClient.sendData("ServerGreeting", "Hello Client".getBytes());
    //We don't want to accept any other clients.
    btServer.disconnect();
}
```

### EasyBluetooth - Client
Once a server is set up and awaiting client connections, we can connect to it using a Bluetooth Client.

```java
// Find the server Bluetooth device.
BluetoothDevice serverDev = BluetoothAdapter.getDefaultAdapter().getRemoteDevice("AA:BB:CC:DD:EE:FF");
IBluetoothClient client = new BluetoothClient.Builder(this.getApplication(), serverDev, ParcelUuid.fromString("00001101-0000-1000-8000-00805F9B34FB"))
    // We want to be notified when connection completes.
    .setConnectionCallback(new SampleConnectionCallback())
    // Let's also get notified if it fails
    .setConnectionFailedListener(new SampleConnectionFailedListener())
    // Receive data from the server
    .setDataCallback(new SampleDataCallback())
    // Be notified when the data is sent to the server or fails to send.
    .setDataSentCallback(new SampleDataSentCallback())
    .build();

// Connect to server
client.connect();
```
### EasyBluetooth - Sample Callbacks and Listeners
```java
import android.util.Log;
import com.newtronlabs.easybluetooth.IBluetoothClient;
import com.newtronlabs.easybluetooth.IBluetoothConnectionCallback;

public class SampleConnectionCallback implements IBluetoothConnectionCallback
{
    public static final String TAG = "easyBt";
    @Override
    public void onConnected(IBluetoothClient bluetoothClient)
    {
        // Connection successful.
        Log.d(TAG, "Connected to: " + bluetoothClient.getNodeId());

        // We can start sending data now.
        bluetoothClient.sendData("ClientGreeting", "Hello Server!!".getBytes());

    }

    @Override
    public void onConnectionSuspended(IBluetoothClient bluetoothClient, int reason)
    {
        // Connection lost.
        if(reason == REASON_CONNECTION_CLOSED)
        {
            Log.d(TAG, "Connection to :" +bluetoothClient.getNodeId() + " ended.");
        }
    }
}
```
```java
import android.util.Log;
import com.newtronlabs.easybluetooth.IBluetoothClient;
import com.newtronlabs.easybluetooth.IBluetoothConnectionFailedListener;

public class SampleConnectionFailedListener implements IBluetoothConnectionFailedListener
{
    public static final String TAG = "easyBt";
    @Override
    public void onConnectionFailed(IBluetoothClient bluetoothClient, int i)
    {
        // Connection attempt failed.
        Log.d(TAG, "Connection Failed!");
    }
}
```
```java
import android.util.Log;
import com.newtronlabs.easybluetooth.IBluetoothDataReceivedCallback;
import com.newtronlabs.easybluetooth.IBluetoothMessageEvent;

public class SampleDataCallback implements IBluetoothDataReceivedCallback
{
    public static final String TAG = "easyBt";

    @Override
    public void onDataReceived(IBluetoothMessageEvent messageEvent)
    {
        // Data was received.
        Log.d(TAG, "Received: " + messageEvent.getTag() + " Data: " + new String(messageEvent.getData()));
    }
}
```
```java
import android.util.Log;
import com.newtronlabs.easybluetooth.IBluetoothClient;
import com.newtronlabs.easybluetooth.IBluetoothDataSentCallback;
import com.newtronlabs.easybluetooth.IBluetoothMessageEvent;

public class SampleDataSentCallback implements IBluetoothDataSentCallback
{
    public static final String TAG = "easyBt";

    @Override
    public void onDataSent(IBluetoothClient bluetoothClient, IBluetoothMessageEvent messageEvent)
    {
        Log.d(TAG, "Data Sent: " + messageEvent.getTag() + " Data: " + new String(messageEvent.getData()));
    }

    @Override
    public void onDataSendFailed(IBluetoothClient bluetoothClient, @SendFailureReason int failureReason)
    {
        if(failureReason == REASON_DATA_FORMAT_INVALID)
        {
            Log.d(TAG, "Failed to send data. Invalid Format");
        }
        else if(failureReason == REASON_REMOTE_CONNECTION_CLOSED)
        {
            Log.d(TAG, "Failed to send data. Connection Lost.");
        }
    }
}
```


## License

Easy Bluetooth binaries and source code can only be used in accordance with Freeware license. That is, freeware may be used without payment, but may not be modified. The developer of Easy Bluetooth retains all rights to change, alter, adapt, and/or distribute the software. Easy Bluetooth is not liable for any damages and/or losses incurred during the use of Easy Bluetooth.

You may not decompile, reverse engineer, pull apart, or otherwise attempt to dissect the source code, algorithm, technique or other information from the binary code of Easy Bluetooth unless it is authorized by existing applicable law and only to the extent authorized by such law. In the event that such a law applies, user may only attempt the foregoing if: (1) user has contacted Newtron Labs to request such information and Newtron Labs has failed to respond in a reasonable time, or (2) reverse engineering is strictly necessary to obtain such information and Newtron Labs has failed to reply. Any information obtained by user from Newtron Labs may be used only in accordance to the terms agreed upon by Newtron Labs and in adherence to Newtron Labs confidentiality policy. Such information supplied by Newtron Labs and received by user shall not be disclosed to a third party or used to create a software substantially similar to the technique or expression of the Newtron Labs Easy Bluetooth software.

You are solely responsible for determining the appropriateness of using Easy Bluetooth and assume any risks associated with Your use of Easy Bluetooth. In no event and under no legal theory, whether in tort (including negligence), contract, or otherwise, unless required by applicable law (such as deliberate and grossly negligent acts) or agreed to in writing, shall Newtron Labs be liable to You for damages, including any direct, indirect, special, incidental, or consequential damages of any character arising as a result of this License or out of the use or inability to use the Easy Bluetooth (including but not limited to damages for loss of goodwill, work stoppage, computer failure or malfunction, or any and all other commercial damages or losses), even if Newtron Labs has been advised of the possibility of such damages.


## Contact

contact@newtronlabs.com
