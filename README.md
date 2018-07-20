# connectLibApp-Usage
connectLibApp Usage about

> step one

```java
     CastManager.register(getApplicationContext());
```

> step two

Create ConnectableDeviceListener and required methods

```java

AlertDialog pairingAlertDialog;
ConnectableDevice mTV;

    private ConnectableDeviceListener deviceListener = new ConnectableDeviceListener() {

        @Override
        public void onPairingRequired(ConnectableDevice device, DeviceService service, PairingType pairingType) {
            Log.d("2ndScreenAPP", "Connected to " + mTV.getIpAddress());

            switch (pairingType) {
                case FIRST_SCREEN:
                    Log.d("2ndScreenAPP", "First Screen");
                    pairingAlertDialog.show();
                    break;

                case PIN_CODE:
                case MIXED:
                    Log.d("2ndScreenAPP", "Pin Code");
                    pairingCodeDialog.show();
                    break;

                case NONE:
                default:
                    break;
            }
        }

        @Override
        public void onConnectionFailed(ConnectableDevice device, ServiceCommandError error) {
            Log.d("2ndScreenAPP", "onConnectFailed");
            connectFailed(mTV);
        }

        @Override
        public void onDeviceReady(ConnectableDevice device) {
            Log.d("2ndScreenAPP", "onPairingSuccess");
            if (pairingAlertDialog.isShowing()) {
                pairingAlertDialog.dismiss();
            }
            if (pairingCodeDialog.isShowing()) {
                pairingCodeDialog.dismiss();
            }
            registerSuccess(mTV);
        }

        @Override
        public void onDeviceDisconnected(ConnectableDevice device) {
            Log.d("2ndScreenAPP", "Device Disconnected");
            connectEnded(mTV);
            connectItem.setTitle("Connect");

            BaseFragment frag = mSectionsPagerAdapter.getFragment(mViewPager.getCurrentItem());
            if (frag != null) {
                Toast.makeText(getApplicationContext(), "Device Disconnected", Toast.LENGTH_SHORT).show();
                frag.disableButtons();
            }
        }

        @Override
        public void onCapabilityUpdated(ConnectableDevice device, List<String> added, List<String> removed) {

        }
    };
    
    void connectFailed(ConnectableDevice device) {
        if (device != null)
            Log.d("2ndScreenAPP", "Failed to connect to " + device.getIpAddress());

        if (mTV != null) {
            mTV.removeListener(deviceListener);
            mTV.disconnect();
            mTV = null;
        }
    }
    
    void registerSuccess(ConnectableDevice device) {
        Log.d("2ndScreenAPP", "successful register");

        BaseFragment frag = mSectionsPagerAdapter.getFragment(mViewPager.getCurrentItem());
        if (frag != null)
            frag.setTv(mTV);
    }
    
    void connectEnded(ConnectableDevice device) {
        if (pairingAlertDialog.isShowing()) {
            pairingAlertDialog.dismiss();
        }
        if (pairingCodeDialog.isShowing()) {
            pairingCodeDialog.dismiss();
        }
        mTV.removeListener(deviceListener);
        mTV = null;
    }
    
```

> step three

```java

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.action_connect:
                hConnectToggle();
                return true;
        }
        return super.onOptionsItemSelected(item);
    }
    //ActionBar Menu tıklamasında bu işlem yapılacak
   public void hConnectToggle() {
        if (!this.isFinishing()) {
            if (mTV != null)
            {
                if (mTV.isConnected())
                    mTV.disconnect();

                connectItem.setTitle("Connect");
                mTV.removeListener(deviceListener);
                mTV = null;
            } else
            {
                dialog.show();
            }
        }
    }
    
```


>step four

SetupPicker prepares necessary dialogue

```java
DevicePicker dp;
android.support.v7.app.AlertDialog dialog;

private void setupPicker() {
        dp = new DevicePicker(this);
        dialog = dp.getPickerDialog("Device List", new AdapterView.OnItemClickListener() {

            @Override
            public void onItemClick(AdapterView<?> arg0, View arg1, int arg2, long arg3) {

                mTV = (ConnectableDevice)arg0.getItemAtPosition(arg2);
                mTV.addListener(deviceListener);
                mTV.setPairingType(null);
                mTV.connect();
                connectItem.setTitle(mTV.getFriendlyName());

                dp.pickDevice(mTV);
            }
        });

        pairingAlertDialog = new AlertDialog.Builder(this)
        .setTitle("Pairing with TV")
        .setMessage("Please confirm the connection on your TV")
        .setPositiveButton("Okay", null)
        .setNegativeButton("Cancel", new DialogInterface.OnClickListener() {

            @Override
            public void onClick(DialogInterface dialog, int which) {
                dp.cancelPicker();

                hConnectToggle();
            }
        })
        .create();

        final EditText input = new EditText(this);
        input.setInputType(InputType.TYPE_CLASS_TEXT);

        final InputMethodManager imm = (InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE);

        pairingCodeDialog = new AlertDialog.Builder(this)
        .setTitle("Enter Pairing Code on TV")
        .setView(input)
        .setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {

            @Override
            public void onClick(DialogInterface arg0, int arg1) {
                if (mTV != null) {
                    String value = input.getText().toString().trim();
                    mTV.sendPairingKey(value);
                    imm.hideSoftInputFromWindow(input.getWindowToken(), 0);
                }
            }
        })
        .setNegativeButton(android.R.string.cancel, new DialogInterface.OnClickListener() {

            @Override
            public void onClick(DialogInterface dialog, int whichButton) {
                dp.cancelPicker();

                hConnectToggle();
                imm.hideSoftInputFromWindow(input.getWindowToken(), 0);
            }
        })
        .create();
    }
```

> step five

Main Method

```java
        setupPicker();

        DiscoveryManager.getInstance().registerDefaultDeviceTypes();
        DiscoveryManager.getInstance().setPairingLevel(PairingLevel.ON);
        DiscoveryManager.getInstance().start();
        
```
