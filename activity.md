```java
  implements ConnectableDeviceListener
```

> Main Activity

```java

        DiscoveryManager.init(getApplicationContext());
        // This step could even happen in your app's delegate
        mDiscoveryManager = DiscoveryManager.getInstance();
        mDiscoveryManager.start();

        final AdapterView.OnItemClickListener selectDevice = new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView adapter, View parent, int position, long id) {
                mDevice = (ConnectableDevice) adapter.getItemAtPosition(position);
                mDevice.addListener(MainActivity.this);
                mDevice.connect();
            }
        };
        setupPlayer();
        
        //tıklama içerisine
        setupPlayer();
        if (mDevice != null)
            castVideo(mDevice)
            
            
```

> setupPlayer()
```java
private void setupPlayer() {
        // 1. Create a default TrackSelector
        Handler mainHandler = new Handler();
        BandwidthMeter bandwidthMeter = new DefaultBandwidthMeter();
        TrackSelection.Factory videoTrackSelectionFactory = new AdaptiveTrackSelection.Factory(bandwidthMeter);
        TrackSelector trackSelector = new DefaultTrackSelector(videoTrackSelectionFactory);

        // 2. Create a default LoadControl
        LoadControl loadControl = new DefaultLoadControl();

        // 3. Create the player
        player = ExoPlayerFactory.newSimpleInstance(this, trackSelector, loadControl);

        simpleExoPlayerView = findViewById(R.id.player_view);
        simpleExoPlayerView.setPlayer(player);
        simpleExoPlayerView.hideController();
        simpleExoPlayerView.setUseController(true);


        // Measures bandwidth during playback. Can be null if not required.
        DefaultBandwidthMeter defaultBandwidthMeter = new DefaultBandwidthMeter(mainHandler, null);
        // Produces DataSource instances through which media data is loaded.
        DataSource.Factory dataSourceFactory = new DefaultDataSourceFactory(this,
                Util.getUserAgent(this, "Exo2"), defaultBandwidthMeter);
        // Produces Extractor instances for parsing the media data.
        ExtractorsFactory extractorsFactory = new DefaultExtractorsFactory();
        // This is the MediaSource representing the media to be played.
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
```

> for device list 
```java
 private void showImage(AdapterView.OnItemClickListener selectDevice) {
        DevicePicker devicePicker = new DevicePicker(this);
        AlertDialog dialog = devicePicker.getPickerDialog("Connectable Devices", selectDevice);
        dialog.show();
    }
```

> castVideo()
```java
private void castVideo(ConnectableDevice device) {
        mediaInfo = new MediaInfo.Builder(mediaURL, mimeType)
                .setTitle(title)
                .setDescription(description)
                .setIcon(iconURL)
                .setSubtitleInfo(subtitles)
                .build();

        MediaPlayer.LaunchListener listener = new MediaPlayer.LaunchListener() {
            @Override
            public void onSuccess(MediaPlayer.MediaLaunchObject object) {
                // save these object references to control media playback
                /*mLaunchSession = object.launchSession;*/
                castMediaControl = object.mediaControl;
                castMediaControl.seek(player.getCurrentPosition(), null);

                // you will want to enable your media control UI elements here
            }

            @Override
            public void onError(ServiceCommandError error) {
                Log.d("App Tag", "Play media failure: " + error);
            }
        };

        mDevice.getMediaPlayer().playMedia(mediaInfo, false, listener);
    }
```
