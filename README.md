# Easy Bluetooth

The EasyBluetooth library allows the fast creation of Bluetooth connections between devices. 

----


## How to Use 

### Setup

Include the below dependencies in your `build.gradle` project.

```gradle
buildscript {
    repositories {
        google()
        jcenter()
        maven { url "https://newtronlabs.jfrog.io/artifactory/libs-release-local" }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.5.2'
        classpath 'com.newtronlabs.android:plugin:4.0.3'
    }
}

allprojects {
    repositories {
        google()
        jcenter()
        maven { url "https://newtronlabs.jfrog.io/artifactory/libs-release-local" }
    }
}

subprojects {
    apply plugin: 'com.newtronlabs.android'
}
```

In the `build.gradle` for your app.

```gradle
dependencies {
    compileOnly 'com.newtronlabs.easybluetooth:easybluetooth:4.0.2'
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
https://gist.github.com/NewtronLabs/216f45db2339e0bc638e7c14a6af9cc8


## Contact

solutions@newtronlabs.com
