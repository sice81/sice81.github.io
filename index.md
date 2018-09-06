# hello

## title1

## title2

```java
package com.exem.test;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hive.conf.HiveConf;
import org.apache.hadoop.hive.metastore.HiveMetaStoreClient;
import org.apache.hadoop.hive.metastore.api.Database;
import org.apache.hadoop.hive.metastore.api.MetaException;
import org.apache.hadoop.security.UserGroupInformation;
import java.security.PrivilegedExceptionAction;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

public class Main {
    static String HIVE_METASTORE_URL = "thrift://exam.com:9083";
    static String PRINCIPAL = "root/exam.com@EXAM.COM";
    static String KEYTAB_FILE = "/root/root.exam.com.keytab";

    static String METASTORE_PRINCIPAL = "hive/_HOST@EXAM.COM";
    static String METASTORE_KEYTAB_FILE = "/etc/hadoop/conf/hive.keytab";

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        conf.set("hadoop.security.authentication", "Kerberos");
        UserGroupInformation.setConfiguration(conf);
        UserGroupInformation.loginUserFromKeytab(PRINCIPAL, KEYTAB_FILE);

        UserGroupInformation ugi = UserGroupInformation.getLoginUser();
        ugi.doAs((PrivilegedExceptionAction<Void>) () -> {
            HiveMetaStoreClient client = Main.getMetaStoreClient();
            System.out.println(client == null ? "client null!" : "client got!");
            client.getAllDatabases().stream().forEach(e -> System.out.println(e));

            Database db = new Database();
            db.setName("testdb03");
            db.setLocationUri("/user/root/testdb03.db");
            client.createDatabase(db);

            return null;
        });
    }

    public static HiveMetaStoreClient getMetaStoreClient() throws Exception {
        try {
            HiveConf conf = new HiveConf();
            conf.setVar(HiveConf.ConfVars.METASTOREURIS, HIVE_METASTORE_URL);
            conf.setVar(HiveConf.ConfVars.METASTORE_USE_THRIFT_SASL, "true");
            conf.setVar(HiveConf.ConfVars.METASTORE_KERBEROS_PRINCIPAL, METASTORE_PRINCIPAL);
            conf.setVar(HiveConf.ConfVars.METASTORE_KERBEROS_KEYTAB_FILE, METASTORE_KEYTAB_FILE);

            return new HiveMetaStoreClient(conf);
        } catch (MetaException e) {
            e.printStackTrace();
            return null;
        }
    }
}

```