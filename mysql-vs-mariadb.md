# MySQL vs MariaDB


MySQL이 Oracle에 인수되고 유료화되면서 반기를 든 기존 개발자 몇몇이 MySQL의 포크(fork) 버전을 독자적으로 개발하면서 오픈소스로 관리하고 있음

나뉘어진지 꽤 오래 ~~년 되었기 때문에 지금은 둘은 아예 다른 DBMS라고 볼 수 있음. 그래도 최상위 SQL 계층에서는 MariaDB와 MySQL은 거의 완벽하게 호환되어 TypeORM같은 ORM 레벨에서는 DB_TYPE을 mariadb로 하든 mysql로 하든 같은 동작을 할 것이다(아마?)

어떤 부분이 다를까