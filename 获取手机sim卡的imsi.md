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
String iccid = tm.getSimSerialNumber();
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

TelephonyManager mTelephonyManager = (TelephonyManager) v.getContext().getSystemService(Context.TELEPHONY_SERVICE);
String subscriberId = mTelephonyManager.getSubscriberId();
Log.d("glen", "getSubscriberId subscriberId = " + subscriberId);

for (int i = 0; i < simCount; i++) {
    SubscriptionInfo info = subscriptionManager.getActiveSubscriptionInfoForSimSlotIndex(i);
    if (null != info) {
        iccid = info.getIccId();
        Log.d("glen", "getActiveSubscriptionInfoForSimSlotIndex info.getIccId = " + iccid);
        int slotIndex = info.getSimSlotIndex();
        Log.d("glen", "getActiveSubscriptionInfoForSimSlotIndex info.getSimSlotIndex = " + slotIndex);
        int subId = info.getSubscriptionId();
        Log.d("glen", "getActiveSubscriptionInfoForSimSlotIndex info.getSubscriptionId = " + subId);
        String imsi = getImsiBySubId(mTelephonyManager,subId);
        Log.d("glen", "getActiveSubscriptionInfoForSimSlotIndex imsi = " + imsi);
    }
}



private String getImsiBySubId(TelephonyManager mTelephonyManager,int subId){
    String imsi = "";
    try {
        Method getImei = mTelephonyManager.getClass().getDeclaredMethod("getSubscriberId",int.class);
        if(getImei != null) {
            imsi = (String)getImei.invoke(mTelephonyManager,subId);
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return imsi;
}



