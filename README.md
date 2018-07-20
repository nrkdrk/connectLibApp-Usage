# connectLibApp-Usage
connectLibApp Usage about
Example usage for Fragment 

> step one

In the Application class
```java
     CastManager.register(getApplicationContext());
```

> step two
Add the things specified to the Project Master Class

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

> Fragment select
```java
        //BaseFragment frag = Fragment();
        if (frag != null)
            frag.setTv(mTV);
```



Main Fragment for BaseFragment

```java
 public class BaseFragment extends Fragment {

    private ConnectableDevice mTv;
    private Launcher launcher;
    private MediaPlayer mediaPlayer;
    private MediaControl mediaControl;
    private TVControl tvControl;
    private VolumeControl volumeControl;
    private ToastControl toastControl;
    private MouseControl mouseControl;
    private TextInputControl textInputControl;
    private PowerControl powerControl;
    private ExternalInputControl externalInputControl;
    private KeyControl keyControl;
    private WebAppLauncher webAppLauncher;
    public Button[] buttons;
    Context mContext;

    public BaseFragment() {}

    @SuppressLint("ValidFragment")
    public BaseFragment(Context context)
    {
        mContext = context;
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);

        setTv(mTv);
    }

    public void setTv(ConnectableDevice tv)
    {
        mTv = tv;

        if (tv == null) {
            launcher = null;
            mediaPlayer = null;
            mediaControl = null;
            tvControl = null;
            volumeControl = null;
            toastControl = null;
            textInputControl = null;
            mouseControl = null;
            externalInputControl = null;
            powerControl = null;
            keyControl = null;
            webAppLauncher = null;

            disableButtons();
        }
        else {
            launcher = mTv.getCapability(Launcher.class);
            mediaPlayer = mTv.getCapability(MediaPlayer.class);
            mediaControl = mTv.getCapability(MediaControl.class);
            tvControl = mTv.getCapability(TVControl.class);
            volumeControl = mTv.getCapability(VolumeControl.class);
            toastControl = mTv.getCapability(ToastControl.class);
            textInputControl = mTv.getCapability(TextInputControl.class);
            mouseControl = mTv.getCapability(MouseControl.class);
            externalInputControl = mTv.getCapability(ExternalInputControl.class);
            powerControl = mTv.getCapability(PowerControl.class);
            keyControl = mTv.getCapability(KeyControl.class);
            webAppLauncher = mTv.getCapability(WebAppLauncher.class);

            enableButtons();
        }
    }

    public void disableButtons() {
        if (buttons != null)
        {
            for (Button button : buttons)
            {
                button.setOnClickListener(null);
                button.setEnabled(false);
            }
        }
    }

    public void enableButtons() {
        if (buttons != null)
        {
            for (Button button : buttons)
                button.setEnabled(true);
        }
    }

    public boolean onKeyDown(int keyCode, KeyEvent event) {
        return false;
    }

    public boolean onKeyUp(int keyCode, KeyEvent event) {
        return false;
    }

    public ConnectableDevice getTv()
    {
        return mTv;
    }

    public Launcher getLauncher() 
    {
        return launcher;
    }

    public MediaPlayer getMediaPlayer()
    {
        return mediaPlayer;
    }

    public MediaControl getMediaControl()
    {
        return mediaControl;
    }

    public VolumeControl getVolumeControl() 
    {
        return volumeControl;
    }

    public TVControl getTVControl() 
    {
        return tvControl;
    }

    public ToastControl getToastControl() 
    {
        return toastControl;
    }

    public TextInputControl getTextInputControl() 
    {
        return textInputControl;
    }

    public MouseControl getMouseControl() 
    {
        return mouseControl;
    }

    public ExternalInputControl getExternalInputControl()
    {
        return externalInputControl;
    }

    public PowerControl getPowerControl() 
    {
        return powerControl;
    }

    public KeyControl getKeyControl() 
    {
        return keyControl;
    }

    public WebAppLauncher getWebAppLauncher() 
    {
        return webAppLauncher;
    }

    public Context getContext()
    {
        return mContext;
    }

    protected void disableButton(final Button button) {
        button.setEnabled(false);
    }
}
```
I am creating a Fragment for example purposes

> DemoFragment

```java
      public class DemoFragment extends BaseFragment  {

    private SimpleExoPlayerView simpleExoPlayerView;
    private SimpleExoPlayer player;
    public TestResponseObject testResponse;
    boolean isPlayingImage = false;
    boolean isPlaying = false;
    private MediaControl mMediaControl = null;
    private PlaylistControl mPlaylistControl = null;
    private Timer refreshTimer;
    public LaunchSession launchSession;
    public final int REFRESH_INTERVAL_MS = (int) TimeUnit.SECONDS.toMillis(1);
    public long totalTimeDuration;
    public static String URL_VIDEO_MP4 = "http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4";
    View rootView;
    String mediaURL = "";
    MediaControl castMediaControl;
    EditText edtUrl;
    Button btnPlay;
    TextView debug;

    public DemoFragment() {};

    @SuppressLint("ValidFragment")
    public DemoFragment(Context context)
    {
        super(context);
        testResponse = new TestResponseObject();
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        setRetainInstance(true);
        rootView = inflater.inflate(
                R.layout.fragment_demo, container, false);

        debug = rootView.findViewById(R.id.debug);
        edtUrl = rootView.findViewById(R.id.edt_video_url);
        btnPlay = rootView.findViewById(R.id.btn_play);
        mediaURL = edtUrl.getText().toString();
        URL_VIDEO_MP4 = edtUrl.getText().toString();

        setupPlayer();
        btnPlay.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if (edtUrl.getText().length() > 0) {
                    mediaURL = edtUrl.getText().toString();
                    setupPlayer();
                }else{
                    Toast.makeText(getContext(), "Enter a content url.", Toast.LENGTH_SHORT).show();
                }
            }
        });

        return rootView;
    }
    private void setupPlayer() {
        Handler mainHandler = new Handler();
        BandwidthMeter bandwidthMeter = new DefaultBandwidthMeter();
        TrackSelection.Factory videoTrackSelectionFactory = new AdaptiveTrackSelection.Factory(bandwidthMeter);
        TrackSelector trackSelector = new DefaultTrackSelector(videoTrackSelectionFactory);
        LoadControl loadControl = new DefaultLoadControl();
        player = ExoPlayerFactory.newSimpleInstance(getContext(), trackSelector, loadControl);

        simpleExoPlayerView = rootView.findViewById(R.id.player_view);
        simpleExoPlayerView.setPlayer(player);
        simpleExoPlayerView.hideController();
        simpleExoPlayerView.setUseController(true);

        DefaultBandwidthMeter defaultBandwidthMeter = new DefaultBandwidthMeter(mainHandler, null);
        DataSource.Factory dataSourceFactory = new DefaultDataSourceFactory(getContext(), Util.getUserAgent(getContext(), "Exo2"), defaultBandwidthMeter);
        ExtractorsFactory extractorsFactory = new DefaultExtractorsFactory();
        HlsMediaSource hlsMediaSource = new HlsMediaSource(Uri.parse(mediaURL), dataSourceFactory, mainHandler, new AdaptiveMediaSourceEventListener() {
            @Override
            public void onLoadStarted(DataSpec dataSpec, int dataType, int trackType, Format trackFormat, int trackSelectionReason, Object trackSelectionData, long mediaStartTimeMs, long mediaEndTimeMs, long elapsedRealtimeMs) {

            }

            @Override
            public void onLoadCompleted(DataSpec dataSpec, int dataType, int trackType, Format trackFormat, int trackSelectionReason, Object trackSelectionData, long mediaStartTimeMs, long mediaEndTimeMs, long elapsedRealtimeMs, long loadDurationMs, long bytesLoaded) {

            }

            @Override
            public void onLoadCanceled(DataSpec dataSpec, int dataType, int trackType, Format trackFormat, int trackSelectionReason, Object trackSelectionData, long mediaStartTimeMs, long mediaEndTimeMs, long elapsedRealtimeMs, long loadDurationMs, long bytesLoaded) {

            }

            @Override
            public void onLoadError(DataSpec dataSpec, int dataType, int trackType, Format trackFormat, int trackSelectionReason, Object trackSelectionData, long mediaStartTimeMs, long mediaEndTimeMs, long elapsedRealtimeMs, long loadDurationMs, long bytesLoaded, IOException error, boolean wasCanceled) {

            }

            @Override
            public void onUpstreamDiscarded(int trackType, long mediaStartTimeMs, long mediaEndTimeMs) {

            }

            @Override
            public void onDownstreamFormatChanged(int trackType, Format trackFormat, int trackSelectionReason, Object trackSelectionData, long mediaTimeMs) {

            }
        });

        player.prepare(hlsMediaSource);
        simpleExoPlayerView.requestFocus();
        player.setPlayWhenReady(true);

        player.addListener(new Player.EventListener() {
            @Override
            public void onTimelineChanged(Timeline timeline, Object manifest, int reason) {

            }

            @Override
            public void onTracksChanged(TrackGroupArray trackGroups, TrackSelectionArray trackSelections) {

            }

            @Override
            public void onLoadingChanged(boolean isLoading) {

            }

            @Override
            public void onPlayerStateChanged(boolean playWhenReady, int playbackState) {
                Log.i("onPlayerStateChanged", playbackState + "");
            }

            @Override
            public void onRepeatModeChanged(int repeatMode) {

            }

            @Override
            public void onShuffleModeEnabledChanged(boolean shuffleModeEnabled) {

            }

            @Override
            public void onPlayerError(ExoPlaybackException error) {

            }

            @Override
            public void onPositionDiscontinuity(int reason) {

            }

            @Override
            public void onPlaybackParametersChanged(PlaybackParameters playbackParameters) {

            }

            @Override
            public void onSeekProcessed() {
                castMediaControl.seek(player.getCurrentPosition(), null);
            }
        });
    }

    @Override
    public void enableButtons() {
        totalTimeDuration = -1;
        if (getTv().hasCapability(MediaPlayer.Play_Video)) {
            btnPlay.setEnabled(true);
            btnPlay.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    playVideo();
                }
            });
        }
        else {
            disableButton(btnPlay);
        }
    }

    private void playVideo() {
        SubtitleInfo.Builder subtitleBuilder = null;
        MediaInfo mediaInfo = new MediaInfo.Builder(URL_VIDEO_MP4, "video/mp4")
                .setTitle("Sintel Trailer")
                .setDescription("Blender Open Movie Project")
                .setIcon(URL_IMAGE_ICON)
                .setSubtitleInfo(subtitleBuilder == null ? null : subtitleBuilder.build())
                .build();

        getMediaPlayer().playMedia(mediaInfo, true, new MediaPlayer.LaunchListener() {
            @Override
            public void onError(ServiceCommandError error) {
                Log.e("Error", "Error playing video", error);
                stopMediaSession();
            }

            public void onSuccess(MediaPlayer.MediaLaunchObject object) {
                launchSession = object.launchSession;
                testResponse = new TestResponseObject(true, TestResponseObject.SuccessCode,
                        TestResponseObject.Play_Video);
                mMediaControl = object.mediaControl;
                mPlaylistControl = object.playlistControl;
                stopUpdating();
                enableMedia();
                isPlaying = true;
            }
        });
    }
    private void stopUpdating() {
        if (refreshTimer == null)
            return;
        refreshTimer.cancel();
        refreshTimer = null;
    }

    private void stopMediaSession() {
        if (launchSession != null) {
            launchSession = null;
            stopUpdating();
            isPlaying = isPlayingImage = false;
        }
    }

    public void enableMedia() {
        if (getTv().hasCapability(MediaControl.PlayState_Subscribe) && !isPlaying) {
            mMediaControl.subscribePlayState(playStateListener);
        } else {
            if (mMediaControl != null) {
                mMediaControl.getDuration(durationListener);
            }
            startUpdating();
        }
    }

    private void startUpdating() {
        if (refreshTimer != null) {
            refreshTimer.cancel();
            refreshTimer = null;
        }
        refreshTimer = new Timer();
        refreshTimer.schedule(new TimerTask() {

            @Override
            public void run() {
                Log.d("LG", "Updating information");
                if (mMediaControl != null && getTv() != null && getTv().hasCapability(MediaControl.Position)) {
                    mMediaControl.getPosition(positionListener);
                }
                if (mMediaControl != null
                        && getTv() != null
                        && getTv().hasCapability(MediaControl.Duration)
                        && !getTv().hasCapability(MediaControl.PlayState_Subscribe)
                        && totalTimeDuration <= 0) {
                    mMediaControl.getDuration(durationListener);
                }
            }
        }, 0, REFRESH_INTERVAL_MS);
    }

    private MediaControl.PositionListener positionListener = new MediaControl.PositionListener() {
        @Override public void onError(ServiceCommandError error) { }
        @Override
        public void onSuccess(Long position) {
            //
        }
    };

    public MediaControl.PlayStateListener playStateListener = new MediaControl.PlayStateListener() {
        @Override
        public void onError(ServiceCommandError error) {
            Log.d("LG", "Playstate Listener error = " + error);
        }

        @Override
        public void onSuccess(MediaControl.PlayStateStatus playState) {
            Log.d("LG", "Playstate changed | playState = " + playState);
            switch (playState) {
                case Playing:
                    startUpdating();
                    if (mMediaControl != null && getTv().hasCapability(MediaControl.Duration)) {
                        mMediaControl.getDuration(durationListener);
                    }
                    break;
                default:
                    stopUpdating();
                    break;
            }
        }
    };

    private MediaControl.DurationListener durationListener = new MediaControl.DurationListener() {
        @Override public void onError(ServiceCommandError error) { }
        @Override
        public void onSuccess(Long duration) {
            totalTimeDuration = duration;
        }
    };
}
```
