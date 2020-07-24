# Android WebView退出之后，音频仍在播放解决方案

```java
//添加一下代码解决问题 webView退出之后音频视频还在播放问题
private AudioManager audioManager;
private AudioManager.OnAudioFocusChangeListener listener;


@Override
protected void onResume() {
    if (audioManager!= null) {
        audioManager.abandonAudioFocus(listener);
        audioManager = null;
    }

    super.onResume();
}

@Override
protected void onPause() {
    audioManager = (AudioManager) mContext.getSystemService(Context.AUDIO_SERVICE);
    listener = new AudioManager.OnAudioFocusChangeListener() {
        @Override
        public void onAudioFocusChange(int focusChange) {
        }
    };
    int result = audioManager.requestAudioFocus(listener, AudioManager.STREAM_MUSIC, AudioManager.AUDIOFOCUS_GAIN_TRANSIENT);

    if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
    }
    super.onPause();
}
```

