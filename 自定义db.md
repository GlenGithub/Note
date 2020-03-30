########################## DbField ##################################
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface DbField {
    String value();
}
########################## DbTable ##################################
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface DbTable {
    String value();
}
########################## StrategyEntity ##################################
@DbTable("t_strategy")
public class StrategyEntity {
    /**
     * ���԰�ID
     */
    private String flowNumber;
    /**
     * ���԰�ǩ��
     */
    private String signature;
    /**
     * ǩ���㷨
     */
    private String algorithm;
    /**
     * ���Զ���汾
     */
    private String definitionVer;
    /**
     * ���԰�����ʱ���
     */
    private String timeStamp;
    /**
     * ���ݰ�����
     */
    private String type;
    /**
     * ���¼��
     */
    private String updateDuration;
    /**
     * Χ���ܿؼ���(JsonArray)
     */
    private String fences;
	/**
     * �ܿز��Լ���(JsonArray)
     */
    private String rules;

    public String getFlowNumber() {
        return flowNumber;
    }

    public void setFlowNumber(String flowNumber) {
        this.flowNumber = flowNumber;
    }

    public String getSignature() {
        return signature;
    }

    public void setSignature(String signature) {
        this.signature = signature;
    }

    public String getAlgorithm() {
        return algorithm;
    }

    public void setAlgorithm(String algorithm) {
        this.algorithm = algorithm;
    }

    public String getDefinitionVer() {
        return definitionVer;
    }

    public void setDefinitionVer(String definitionVer) {
        this.definitionVer = definitionVer;
    }

    public String getTimeStamp() {
        return timeStamp;
    }

    public void setTimeStamp(String timeStamp) {
        this.timeStamp = timeStamp;
    }
	
	public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getUpdateDuration() {
        return updateDuration;
    }

    public void setUpdateDuration(String updateDuration) {
        this.updateDuration = updateDuration;
    }

    public String getFences() {
        return fences;
    }

    public void setFences(String fences) {
        this.fences = fences;
    }

    public String getRules() {
        return rules;
    }

    public void setRules(String rules) {
        this.rules = rules;
    }
	
	@Override
    public String toString() {
        JSONObject object = new JSONObject();
        try {
            object.put("flowNumber", flowNumber);
            object.put("signature", signature);
            object.put("algorithm", algorithm);
            object.put("definitionVer", definitionVer);
            object.put("timeStamp", timeStamp);
            object.put("type", type);
            object.put("updateDuration", updateDuration);
            object.put("fences", new JSONArray(fences));
            object.put("rules", new JSONArray(rules));
        } catch (Exception e) {
            e.printStackTrace();
            Log.d("StrategyEntity", "toString Exception");
        }
        Log.d("StrategyEntity", object.toString());
        return object.toString();
    }
}
########################## IBaseDao ##################################
public interface IBaseDao<T> {
    /**
     * ��������
     *
     * @param entity
     * @return
     */
    long insert(T entity);

    /**
     * ��������
     *
     * @param entity
     * @return
     */
    long update(T entity, T where);

    /**
     * ɾ������
     *
     * @param where
     * @return
     */
    int delete(T where);
	/**
     * ��ѯ����
     *
     * @param where
     * @return
     */
    List<T> query(T where);

    List<T> query(T where, String orderBy, Integer startIndex, Integer limit);

    List<T> query(String sql);
}
########################## BaseDao ##################################
@SuppressWarnings("all")
public class BaseDao<T> implements IBaseDao<T> {

    /**
     * �������ݿ������
     */
    private SQLiteDatabase sqLiteDatabase;

    /**
     * ����
     */
    private String tableName;

    /**
     * ���в������ݿ�����Ӧ��java����
     */
    private Class<T> entityClass;
	/**
     * ��ʶ��������ʾ�Ƿ�������ʼ������
     */
    private boolean isInit = false;

    /**
     * ����һ������ռ�(key - �ֶ��� ��value - ��Ա����)
     */
    private HashMap<String, Field> cacheMap;

    /**
     * @param sqLiteDatabase
     * @param entityClass
     */
    protected boolean init(SQLiteDatabase sqLiteDatabase, Class<T> entityClass) {
        this.sqLiteDatabase = sqLiteDatabase;
        this.entityClass = entityClass;
		//���Ը��ݴ����entityClass������������,ֻ��Ҫ����һ��
        if (!isInit) {
            //�Զ�����
            //ȡ������
            if (entityClass.getAnnotation(DbTable.class) == null) {
                //���䵽����
                tableName = entityClass.getSimpleName();
            } else {
                tableName = entityClass.getAnnotation(DbTable.class).value();
            }

            if (!sqLiteDatabase.isOpen()) {
                return false;
            }
			//ִ�б����
            //create table if not exists t_student(_id integer,age varchar(20),sex varchar(20),hight varchar(20))
            String createTableSql = getCreateTableSql();
            sqLiteDatabase.execSQL(createTableSql);
            cacheMap = new HashMap<>();
            initCacheMap();
            isInit = true;
        }
        return isInit;
    }
	
	public String getCreateTableSql() {
        //create table if not exists t_student(_id integer,age varchar(20),sex varchar(20),hight varchar(20))
        StringBuffer stringBuffer = new StringBuffer();
        stringBuffer.append("create table if not exists ");
        stringBuffer.append(tableName + "(");
        //����õ����г�Ա����
        Field[] fields = entityClass.getDeclaredFields();
        for (Field field : fields) {
            //�õ���Ա����
            Class type = field.getType();
            if (field.getAnnotation(DbField.class) != null) {
			if (type == String.class) {
                    stringBuffer.append(field.getAnnotation(DbField.class).value() + " TEXT,");
                } else if (type == Integer.class) {
                    stringBuffer.append(field.getAnnotation(DbField.class).value() + " INTEGER,");
                } else if (type == Long.class) {
                    stringBuffer.append(field.getAnnotation(DbField.class).value() + " BIGINT,");
                } else if (type == Double.class) {
                    stringBuffer.append(field.getAnnotation(DbField.class).value() + " DOUBLE,");
                } else if (type == byte[].class) {
                    stringBuffer.append(field.getAnnotation(DbField.class).value() + " BLOB,");
                } else {
                    //��֧�ֵ�����
                    continue;
                }
            } else {
			if (type == String.class) {
                    stringBuffer.append(field.getName() + " TEXT,");
                } else if (type == Integer.class) {
                    stringBuffer.append(field.getName() + " INTEGER,");
                } else if (type == Long.class) {
                    stringBuffer.append(field.getName() + " BIGINT,");
                } else if (type == Double.class) {
                    stringBuffer.append(field.getName() + " DOUBLE,");
                } else if (type == byte[].class) {
                    stringBuffer.append(field.getName() + " BLOB,");
                } else {
                    //��֧�ֵ�����
                    continue;
                }
            }
        }
		int length = stringBuffer.length();
        if (stringBuffer.charAt(length - 1) == ',') {
            stringBuffer.deleteCharAt(length - 1);
        }
        stringBuffer.append(")");
        Log.d("basedao", stringBuffer.toString());
        return stringBuffer.toString();
    }
	
	/**
     * ����
     */
    private void initCacheMap() {
        Cursor cursor = null;
        try {
            //ȡ����������
            String sql = "select * from " + tableName + " limit 1,0";
            cursor = sqLiteDatabase.rawQuery(sql, null);
            String[] columnNames = cursor.getColumnNames();
            //��ȡ���г�Ա����
            Field[] columnFields = entityClass.getDeclaredFields();
			//�������ֶεķ���Ȩ�޴�
            for (Field field : columnFields) {
                field.setAccessible(true);
            }

            //����������������ӳ��
            for (String columnName : columnNames) {
                Field columnField = null;
                for (Field field : columnFields) {
                    String fieldName = null;
                    if (field.getAnnotation(DbField.class) != null) {
                        fieldName = field.getAnnotation(DbField.class).value();
						} else {
                        fieldName = field.getName();
                    }

                    if (columnName.equals(fieldName)) {
                        columnField = field;
                        break;
                    }
                }

                if (columnField != null) {
                    cacheMap.put(columnName, columnField);
                }
            }
			} catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (cursor != null) {
                cursor.close();
                cursor = null;
            }
        }
    }
	
	@Override
    public long insert(T entity) {
        long result = -1;
        try {
            Map<String, String> map = getValues(entity);
            //������ת�Ƶ�ContentValues��
            ContentValues values = getContentValues(map);
            //��ʼ��������
            result = sqLiteDatabase.insert(tableName, null, values);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
	
	@Override
    public long update(T entity, T where) {
        long result = -1;
        try {
            Map<String, String> map = getValues(entity);
            ContentValues values = getContentValues(map);

            Map<String, String> whereMap = getValues(where);
            Condition condition = new Condition(whereMap);
            result = sqLiteDatabase.update(tableName, values, condition.whereClause, condition.whereArgs);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
	
	@Override
    public int delete(T where) {
        int result = -1;
        try {
            Map<String, String> whereMap = getValues(where);
            Condition condition = new Condition(whereMap);
            result = sqLiteDatabase.delete(tableName, condition.whereClause, condition.whereArgs);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
	
	@Override
    public List<T> query(T where) {
        List<T> list = null;
        Cursor cursor = null;
        try {
            Map<String, String> whereMap = getValues(where);
            Condition condition = new Condition(whereMap);
            cursor = sqLiteDatabase.query(tableName, null, condition.whereClause, condition.whereArgs, null, null, null);
            list = getDataList(cursor);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (cursor != null) {
                cursor.close();
                cursor = null;
            }
        }
        return list;
    }
	
	@Override
    public List<T> query(T where, String orderBy, Integer startIndex, Integer limit) {
        List<T> list = null;
        Cursor cursor = null;
        try {
            String limitStr = "limit " + startIndex + "," + limit;
            Map<String, String> whereMap = getValues(where);
            Condition condition = new Condition(whereMap);
            cursor = sqLiteDatabase.query(tableName, null, condition.whereClause, condition.whereArgs, null, null, orderBy, limitStr);
            list = getDataList(cursor);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (cursor != null) {
                cursor.close();
                cursor = null;
            }
        }
        return null;
    }
	
	@Override
    public List<T> query(String sql) {
        List<T> list = null;
        Cursor cursor = null;
        try {
            cursor = sqLiteDatabase.rawQuery(sql, null);
            list = getDataList(cursor);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (cursor != null) {
                cursor.close();
                cursor = null;
            }
        }

        return list;
    }
	
	/**
     * �����Ƿ��������
     */
    public boolean isExistedData() {
        String sql = "select * from " + tableName;
        List<T> list = query(sql);
        if (list != null && list.size() > 0) {
            return true;
        }
        return false;
    }
	
	/**
     * ͨ���α꣬����������ת������
     */
    private List<T> getDataList(Cursor cursor) throws IllegalAccessException, InstantiationException {
        if (cursor != null) {
            List<T> result = new ArrayList<>();
            //�����α�
            while (cursor.moveToNext()) {
                //��ȡ��ǰnew����ķ��͵ĸ�������
                ParameterizedType pt = (ParameterizedType) this.getClass().getGenericSuperclass();
                //��ȡ��һ�����Ͳ�������ʵ����
                Class<T> clazz = (Class<T>) pt.getActualTypeArguments()[0];
                T item = clazz.newInstance();
				//�������ֶΣ�ʹ���α�һ����ȡֵ����ֵ���´����Ķ���
                Iterator<String> iterator = cacheMap.keySet().iterator();
                while (iterator.hasNext()) {
                    //�ҵ����ֶ�
                    String columnName = iterator.next();
                    //�ҵ����ֶε�������
                    Field field = cacheMap.get(columnName);
                    //�������������ͣ�ʹ���α��ȡ���е�ֵ
                    Object val = null;
                    Class<?> fieldType = field.getType();
                    if (fieldType == String.class) {
					val = cursor.getString(cursor.getColumnIndex(columnName));
                    } else if (fieldType == int.class || fieldType == Integer.class) {
                        val = cursor.getInt(cursor.getColumnIndex(columnName));
                    } else if (fieldType == double.class || fieldType == Double.class) {
                        val = cursor.getDouble(cursor.getColumnIndex(columnName));
                    } else if (fieldType == float.class || fieldType == Float.class) {
                        val = cursor.getFloat(cursor.getColumnIndex(columnName));
                    }
                    //������������Ը�ֵ
                    field.set(item, val);
                }
                //��������ӵ�����
                result.add(item);
				}
            return result;
        }
        return null;
    }
	
	 /**
     * ��ȡ����Ҫ�洢������
     *
     * @param entity
     * @return
     */
    private Map<String, String> getValues(T entity) {
        HashMap<String, String> map = new HashMap<>();
        //�������г�Ա����
        Iterator<Field> fieldIterator = cacheMap.values().iterator();
        while (fieldIterator.hasNext()) {
            Field field = fieldIterator.next();
            field.setAccessible(true);
            //��ȡ��Ա������ֵ
            try {
                //��ȡ�������ݵ�ֵ
                Object object = field.get(entity);
                if (object == null) {
                    continue;
                }
				String value = object.toString();
                //��ȡ����
                String key = null;
                if (field.getAnnotation(DbField.class) != null) {
                    key = field.getAnnotation(DbField.class).value();
                } else {
                    key = field.getName();
                }

                if (!TextUtils.isEmpty(key) && !TextUtils.isEmpty(value)) {
                    map.put(key, value);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return map;
    }
	
	/**
     * ���ݴ洢��ContentValues
     *
     * @param map
     */
    private ContentValues getContentValues(Map<String, String> map) {
        ContentValues contentValues = new ContentValues();
        Set keys = map.keySet();
        Iterator<String> iterator = keys.iterator();
        while (iterator.hasNext()) {
            String key = iterator.next();
            String value = map.get(key);
            if (value != null) {
                contentValues.put(key, value);
            }
        }
        return contentValues;
    }
	
	 class Condition {

        String whereClause;
        String[] whereArgs;

        public Condition(Map<String, String> whereMap) {
            StringBuilder sb = new StringBuilder();
            List<String> list = new ArrayList<>();

            for (Map.Entry<String, String> entry : whereMap.entrySet()) {
                if (!TextUtils.isEmpty(entry.getValue())) {
                    sb.append("and " + entry.getKey() + "=? ");
					list.add(entry.getValue());
                }
            }
            this.whereClause = sb.delete(0, 4).toString();
            this.whereArgs = list.toArray(new String[list.size()]);
        }
    }
}
########################## BaseDaoFactory ##################################
public class BaseDaoFactory {
    private static final BaseDaoFactory instance = new BaseDaoFactory();

    private SQLiteDatabase database;

    public BaseDaoFactory() {
        DBCipherHelper dbCipherHelper = new DBCipherHelper(AEmmApplicaton.getContext());
        database = dbCipherHelper.getWritableDatabase(BuildConfig.DB_PWD);
    }

    public static BaseDaoFactory getInstance() {
        return instance;
    }

    /**
     * ��������BaseDao����
     */
    public <T extends BaseDao<M>, M> T getBaseDao(Class<T> clazz, Class<M> entity) {
        T baseDao = null;
        try {
            baseDao = clazz.newInstance();
            baseDao.init(database, entity);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return baseDao;
    }
}
########################## DBCipherHelper ##################################
public class DBCipherHelper extends SQLiteOpenHelper {

    public static final String TAG = DBCipherHelper.class.getSimpleName();
    public static final String DB_NAME = "cipher.db";
    public static final int DB_VERSION = 1;
    public SQLiteDatabase database;

    public DBCipherHelper(Context context) {
        this(context, DB_NAME, null, DB_VERSION);
    }

    public DBCipherHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
        //����SO
        SQLiteDatabase.loadLibs(context);
    }

    public SQLiteDatabase getDatabase() {
        return database;
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        database = db;
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
}
########################## StrategyDao ##################################
/**
 * ֻ�趨�壬����ʵ��
 *
 * @author Administrator
 * @date 2018/10/19
 */

public class StrategyDao extends BaseDao<StrategyEntity> {
}
