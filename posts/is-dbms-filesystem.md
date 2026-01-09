# DBMS는 파일 시스템의 일종으로 볼 수 있을까?

*__추후 더 작성 예정__*

[운영체제 공부](os.md)를 하다보니 파일시스템은 DBMS와 상당히 닮아있음을 느꼈다. 이유는
1. 자체 인덱스 구조를 가짐
    - DBMS: 데이터 블럭을 관리하기 위한 B-Tree
    - 파일시스템: inode를 관리하기 위한 B-Tree
2. DBMS와 파일 시스템은 모두 버퍼가 있어 캐싱을 한다.
    - DBMS의 Buffer Pool vs OS의 Page Cache: 메모리 관리 전략의 유사성
3. DBMS과 파일시스템 모두 페이지를 할당해 사용하고 반환할 수 있다.
4. 데이터 영속성을 위해 트랜잭션을 지원한다.
    - Atomic Operation: 둘 다 All-or-Nothing 보장
5. DBMS의 Lock Manager vs OS의 Synchronization Primitives: 동시성 제어 메커니즘
6. WAL (Write-Ahead Logging) vs OS의 Journaling File System: 복구 메커니즘의 유사성
7. 디스크에 저장된 데이터를 사용하기 좋게 가공해서 제공한다
    - DBMS: Database, Table(row, column)
    - 파일 시스템: 계층형 디렉토리

좀 찾아보니 비슷한 생각들이 있었다.   
https://www.geeksforgeeks.org/dbms/difference-between-file-system-and-dbms/
https://www.interviewbit.com/blog/file-system-vs-dbms/

DBMS는 파일시스템 위에 구현된 또다른 소프트웨어로, 파일 시스템보다 상위 개념이었다. 비슷하게 보이는 까닭은 DBMS가 파일시스템이 하는 기능들을 자기들의 목적에 맞게 재정의해서 사용하기 때문이었던 것 같다. 근데 DBMS가 파일시스템 자체가 되는 상황도 있는듯?

좀 더 찾아보고 어떻게 다른지 적어야겠다.