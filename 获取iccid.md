public void getIccid(View v) {
        try {
            TelephonyManager tm = (TelephonyManager) getSystemService(Context.TELEPHONY_SERVICE);
            if (ActivityCompat.checkSelfPermission(this, Manifest.permission.READ_PHONE_STATE) != PackageManager.PERMISSION_GRANTED) {
                // TODO: Consider calling
                //    ActivityCompat#requestPermissions
                // here to request the missing permissions, and then overriding
                //   public void onRequestPermissionsResult(int requestCode, String[] permissions,
                //                                          int[] grantResults)
                // to handle the case where the user grants the permission. See the documentation
                // for ActivityCompat#requestPermissions for more details.
                return;
            }
			iccid = tm.getSimSerialNumber();
            Log.d("glen", "iccid1 = " + iccid);

            SubscriptionManager subscriptionManager = (SubscriptionManager) getSystemService(Context.TELEPHONY_SUBSCRIPTION_SERVICE);
            List<SubscriptionInfo> subscriptionInfos = subscriptionManager.getActiveSubscriptionInfoList();
            if (null != subscriptionInfos && subscriptionInfos.size() > 0) {
                for (SubscriptionInfo info : subscriptionInfos) {
                    if (null != info) {
                        Log.d("glen", "getActiveSubscriptionInfoList info.getIccId = " + info.getIccId());
                    }
                }
            }
			int simCount = subscriptionManager.getActiveSubscriptionInfoCount();
            Log.d("glen", "simCount = " + simCount);
            int simCountMax = subscriptionManager.getActiveSubscriptionInfoCountMax();
            Log.d("glen", "simCountMax = " + simCountMax);

            for (int i = 0; i < simCount; i++) {
                SubscriptionInfo info = subscriptionManager.getActiveSubscriptionInfo(i);
                if (null != info) {
                    Log.d("glen", "getActiveSubscriptionInfo info.getIccId = " + info.getIccId());
                    iccid = info.getIccId();
                }
            }
			for (int i = 0; i < simCount; i++) {
                SubscriptionInfo info = subscriptionManager.getActiveSubscriptionInfoForSimSlotIndex(i);
                if (null != info) {
                    Log.d("glen", "getActiveSubscriptionInfoForSimSlotIndex info.getIccId = " + info.getIccId());
                    iccid = info.getIccId();
                }
            }

            int defaultDataSubscriptionId = 0;
			if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.N) {
                defaultDataSubscriptionId = SubscriptionManager.getDefaultDataSubscriptionId();
                Log.d("glen", "defaultDataSubscriptionId = " + defaultDataSubscriptionId);

                int defaultSmsSubscriptionId = SubscriptionManager.getDefaultSmsSubscriptionId();
                Log.d("glen", "defaultSmsSubscriptionId = " + defaultSmsSubscriptionId);

                int defaultSubscriptionId = SubscriptionManager.getDefaultSubscriptionId();
                Log.d("glen", "defaultSubscriptionId = " + defaultSubscriptionId);

                int defaultVoiceSubscriptionId = SubscriptionManager.getDefaultVoiceSubscriptionId();
                Log.d("glen", "defaultVoiceSubscriptionId = " + defaultVoiceSubscriptionId);
            }
			SubscriptionInfo info = subscriptionManager.getActiveSubscriptionInfo(defaultDataSubscriptionId);
            if (null != info) {
                Log.d("glen", "getActiveSubscriptionInfo info.getIccId() = " + info.getIccId());
            }
            File file = getExternalCacheDir();
            Log.d("glen", "getExternalCacheDir file.getAbsolutePath() = " + file.getAbsolutePath());

            File[] files = getExternalCacheDirs();
            for (File item : files) {
                Log.d("glen", "getExternalCacheDirs item.getAbsolutePath() = " + item.getAbsolutePath());
            }
			File pictureFile = getExternalFilesDir(android.os.Environment.DIRECTORY_PICTURES);
            Log.d("glen", "getExternalFilesDir pictureFile.getAbsolutePath() = " + pictureFile.getAbsolutePath());

            tv_result.setText("iccid:" + iccid);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
	
	********************************************************************************
	
/**
     * 3秒内5次点击执行某个操作
     */
    private void execActonAtTimes() {
        if (counts == null) {
            counts = new long[5];
        }
        System.arraycopy(counts, 1, counts, 0, counts.length - 1);
        counts[counts.length - 1] = SystemClock.uptimeMillis();
        if (counts[0] > SystemClock.uptimeMillis() - 3000) {
            Log.d("glen", "你已经在3秒内连续点击了" + counts.length + "次了");
            counts = null;
            openTestPage();
        }
    }
	
	********************************************************************************
	
	public class C3Utils {
    public static String getStoragePath(Context context, boolean is_removable) {
        List<StorageInfo> list = listAllStorage(context);

        for (int i = 0; i < list.size(); i++) {
            String path = list.get(i).path;
            String state = list.get(i).state;
            Log.v("glen", "getStoragePath() -> path = "+path+", = "+state);
            boolean removeable = list.get(i).isRemoveable;
            if (is_removable == removeable && list.get(i).state.equals("mounted")) {
                Log.v("glen", "getStoragePath() -> path = "+path);
                return path;
            }
        }
        return "";
    }
	private static List<StorageInfo> listAllStorage(Context mContext ) {
        ArrayList<StorageInfo> storages = new ArrayList<StorageInfo>();
        StorageManager storageManager = (StorageManager) mContext.getSystemService(Context.STORAGE_SERVICE);
        try {
            Class<?>[] paramClasses = {};
            Method getVolumeList = StorageManager.class.getMethod("getVolumeList", paramClasses);
            Object[] params = {};
            Object[] invokes = (Object[]) getVolumeList.invoke(storageManager, params);
            if (invokes != null) {
			StorageInfo info = null;
                for (int i = 0; i < invokes.length; i++) {
                    Object obj = invokes[i];
                    Method getPath = obj.getClass().getMethod("getPath");
                    String path = (String) getPath.invoke(obj, new Object[0]);
                    Log.d("glen","getPath() -> " +path);
                    info = new StorageInfo(path);

                    Method getVolumeState = StorageManager.class.getMethod("getVolumeState", String.class);
                    String state = (String) getVolumeState.invoke(storageManager, info.path);
                    info.state = state;
                    Method isRemovable = obj.getClass().getMethod("isRemovable");
                    info.isRemoveable = ((Boolean) isRemovable.invoke(obj, new Object[0])).booleanValue();
                    storages.add(info);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
		storages.trimToSize();
        return storages;
    }
	
	/**
     * 删除文件接口
     * @param file 文件名
     */
    public static void DeleteFile(File file) {
        Log.d("glen","DeleteFile："+file.getAbsolutePath());
        if (!file.exists()) {
            Log.d("glen","文件不存在："+file.getAbsolutePath());
            return;
        }
        if (file.isFile()) {
            boolean success = file.delete();
            Log.d("glen","delete file path: "+file.getAbsolutePath()+",status = "+ success);
            return;
        }
		if (file.isDirectory()) {
            Log.d("glen","file.isDirectory()");
            File[] childFile = file.listFiles();
            if (childFile == null || childFile.length == 0) {
                Log.d("glen","delete dir path: "+file.getAbsolutePath());
                file.delete();
                return;
            }
            for (File f : childFile) {
                DeleteFile(f);
            }
            file.delete();
        }

    }
}


//test
//获取用户ID
        String userId = Settings.System.getString(getContentResolver(), "user_id");
        String delPath = C3Utils.getStoragePath(this, true) + "/DCIM/" + userId + "/";
        Log.v("glen", "cleanDataOnC3() -> delPath = " + delPath);
        C3Utils.DeleteFile(new File("/storage/sdcard0/DCIM/" + userId + "/"));
        C3Utils.DeleteFile(new File("/storage/sdcard1/DCIM/" + userId + "/"));
        C3Utils.DeleteFile(new File(delPath));