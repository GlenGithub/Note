public static String getAlgorithm(Context context, String packageName,String algorithm) {
        StringBuilder sb = new StringBuilder();
        try {
            PackageInfo packageInfo = context.getPackageManager().getPackageInfo(packageName, PackageManager.GET_SIGNATURES);
            byte[] cert = packageInfo.signatures[0].toByteArray();
            MessageDigest md = MessageDigest.getInstance(algorithm);
            byte[] publicKey = md.digest(cert);
			for(int i = 0; i < publicKey.length; i++) {
                String appendString = Integer.toHexString(0xFF & publicKey[i]).toUpperCase();
                if(appendString.length() == 1){
                    sb.append("0");
                }
                sb.append(appendString);
                if(i != publicKey.length -1){
                    sb.append(":");
                }
            }
			String result = sb.toString();
            Log.d("glen", packageName + " "+algorithm +" = " + result);
        } catch (PackageManager.NameNotFoundException e) {
            e.printStackTrace();
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }
        return sb.toString();
}
	
	
	public static String getAlgorithm( byte[] cert,String algorithm) {
        StringBuilder sb = new StringBuilder();
        try {
            MessageDigest md = MessageDigest.getInstance(algorithm);
            byte[] publicKey = md.digest(cert);
			for(int i = 0; i < publicKey.length; i++) {
                String appendString = Integer.toHexString(0xFF & publicKey[i]).toUpperCase();
                if(appendString.length() == 1){
                    sb.append("0");
                }
                sb.append(appendString);
                if(i != publicKey.length -1){
                    sb.append(":");
                }
            }
            String result = sb.toString();
			Log.d("glen", ""+algorithm +" = " + result);
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }
        return sb.toString();
    }
	
##################################################################################

public void getCertSignInfo(View v){
        getAlgorithm(this,"com.test.app1","MD5");
        getAlgorithm(this,"com.test.app1","SHA1");
        getAlgorithm(this,"com.test.app1","SHA-256");

        try{
            InputStream rootIs = getAssets().open("cert/CERT.RSA");
            byte[] rootByteArray = InputStreamToByteArray(rootIs);
            X509Certificate x509Cert = getCACert(rootByteArray);
            byte[] cert = x509Cert.getSignature();
            getAlgorithm(cert,"MD5");
            getAlgorithm(cert,"SHA1");
            getAlgorithm(cert,"SHA-256");
        }catch (Exception e){
            e.printStackTrace();
        }

}
