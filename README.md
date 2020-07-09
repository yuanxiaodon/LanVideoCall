# LanVideoCall
android 局域网可视对讲
使用步骤:
一：将arr包导入app工程的libs包中

二：build.gradle中添加一下依赖
    implementation fileTree(include: ['*.jar','*.aar'], dir: 'libs')
    
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'com.google.code.gson:gson:2.8.5'
    implementation 'org.ligboy.retrofit2:converter-fastjson-android:2.1.0'
    implementation 'com.yanzhenjie:permission:2.0.0-rc4'
    
三:初始化aar库

   NettyCore.me().init(targetIp, 8888, 9999);
   
四:初始化可视对讲
   boolean isSend = true;
   
   private SurfaceView localView;
   
   private SurfaceView remoteView;
   
    
    
   TenetCore.me().initCall(new KMessageCall() {
                                    @Override
                                    public void onTcpClientReceive(Object msg) {
                                        Log.e(TAG, "onMessageResponse:" + msg);
                                    }
                                    @Override
                                    public void onTcpServerReceive(Object msg) {
                                        Log.e(TAG, "onMessageResponse:" + msg);

                                    }
                                    @Override
                                    public void onUdpServerReceive(Object msg) {
                                        synchronized (msg) {
                                            DatagramPacket packet = (DatagramPacket) msg;
                                            ByteBuf buf = (ByteBuf) packet.copy().content(); //字节缓冲区
                                            byte[] req = new byte[buf.readableBytes()];
                                            buf.readBytes(req);
                                            try {
                                                String receiveMsg = new String(req, "UTF-8");
                                                Log.e(TAG, "接收消息" + receiveMsg);
                                                KMessage message = JSONObject.parseObject(receiveMsg, KMessage.class);
                                                switch (message.getMsgtype()) {
                                                    case KMessage.MES_TYPE_NOMAL:
                                                        Log.e(TAG, "接收普通消息" + message.getMsgBody());
                                                        break;
                                                    case KMessage.MES_TYPE_VIDEO:
                                                        Log.e(TAG, "接收视频消息" + message.getFrame().length);
                                                        TenetCore.me().playVideo(message.getFrame());
                                                        break;
                                                    case KMessage.MES_TYPE_AUDIO:
                                                        Log.e(TAG, "接收音频消息" + message.getFrame().length);
                                                        TenetCore.me().playAudio(message.getFrame(), message.getFrame().length);
                                                        break;
                                                }
                                            } catch (UnsupportedEncodingException e) {
                                                e.printStackTrace();
                                            }
                                        }
                                    }
                                },
                localView, remoteView, new BaseCallBack() {
                    @Override
                    public void audioEncode(byte[] audioBytes) {
                        if (isSend) {
                            NettyCore.me().udpSend(audioBytes, KMessage.MES_TYPE_AUDIO);
                        }
                    }
                    @Override
                    public void cameraEncode(byte[] cameraBytes) {
                        if (isSend) {
                            Log.e(TAG, "发送视频消息");
                            NettyCore.me().udpSend(cameraBytes, KMessage.MES_TYPE_VIDEO);
                        }
                    }

                    @Override
                    public void cameraDecode(byte[] cameraBytes) {
                        Log.e(TAG, "222解码后视频长度" + cameraBytes.length+",content:"+ DataFormatUtil.bytes2HexString(cameraBytes));
                    }
                });
                
 五:启动编码
 
    TenetCore.me().startEncode();
    
    
 六:停止编码
 
      TenetCore.me().stopEncode();
      
      
 说明:想要修改源码的自己将aar包的代码copy出来使用就可以了
