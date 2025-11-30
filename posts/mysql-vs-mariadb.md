# MySQL vs MariaDB

*__추후 더 작성 예정__*

MySQL이 Oracle에 인수되고 유료화되면서 반기를 든 기존 개발자 몇몇이 MySQL의 포크(fork) 버전을 독자적으로 개발하면서 오픈소스로 관리하고 있음

나뉘어진지 꽤 오래되어 지금은 둘은 아예 다른 DBMS로 간주할 수 있다. 기본적인 SQL 계층에서는 MariaDB와 MySQL이 상당 부분 호환되지만, 각자 독자적인 기능과 최적화가 추가되면서 버전이 올라갈수록 호환성에 차이가 발생한다. TypeORM과 같은 ORM에서도 DB_TYPE을 `mysql`과 `mariadb`로 구분하여 처리하며, 일부 기능이나 쿼리에서 차이가 있을 수 있다. (참고: [MariaDB vs MySQL Compatibility](https://mariadb.com/kb/en/mariadb-vs-mysql-compatibility/), [TypeORM Docs](https://typeorm.io/data-source-options))

어떤 부분이 다를까