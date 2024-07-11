```bash
# über Konsole einloggen
mysql -u user01 -pmypa55 -h 192.168.178.1 -P 3306 # Port und Host teilweise nicht notwendig

# Ein einzelnes SQL Kommando ausführen
echo "show databases;" | mysql -uduffman -h 192.168.56.4 -psaysoyeah

# In DB auf Host 192.168.88.4 als root einloggen und sql Anweisungen aus sql/beer.sql ausführen
mysql -uroot -h 192.168.88.4 -pSQLp4ss < /sql/beer.sql

# MySQL User anzeigen
SELECT user FROM mysql.user;
```
