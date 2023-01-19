---
title: MySQL 시작하기
date: 2023-01-19 22:07:14
tags: [MySQL]
---

# MySQL 시작하기

여태 애써 무시하던 database, 나랑은 상관없겠지(제발) 라는 마인드로 살아왔지만 실무에서 꽤 자주 마주하게 되고, data engineer들의 외계어같은 용어들을 이해할 수가 없더라. 깊은 반성과 함께 **이해하려고 노력**해보자. 먼저 알아야할 키워드는 DBMS, SQL이다.

**DBMS**(Database Management System)는 이름 그대로 DB 관리 시스템이다. 크게 계층형, 네트워크형, 관계형, 객체지향형으로 구분되는 것 같은데, 현재 관계형 DBMS(이하 **RDBMS**)가 대세라고 한다. [링크](https://db-engines.com/en/ranking)에서 현재 어떤 DBMS가 인기가 많은지 확인할 수 있는데, 작성일 기준 많은 RDBMS들이 상위에 랭크 중이고 Oracle과 MySQL이 1위를 두고 경쟁중이다.

![Untitled](resource/MySQL/Untitled.png)

**SQL**(Structured Query Language)은 RDBMS에서 사용하는 언어라고 한다. RDBMS의 시대가 지면 사장될 언어로 보이지만, ANSI 및 ISO에서 결정하는 표준 SQL이 있는 만큼 믿고 사용해도 될 듯 하다. 실제로 RDBMS 이후 객체지향형, 객체관계형 DBMS들이 등장했지만.. 랭킹은 거짓말을 하지 않는다. 그럼 SQL에는 어떤 명령어들이 있을까?

- **DDL**(Data Definition Language) - `CREATE`, `ALTER`, `DROP`, `TRUNCATE`
- **DML**(Data Manipulation Language) - `INSERT`, `UPDATE`, `DELETE`, `SELECT`
- **DCL**(Data Control Language) - `GRANT`, `REVOKE`

크게 3개 정도로 구분되는데 이들을 특별히 구분하기 보다는 실제로 사용해보고 자연스럽게 숙지하는 것이 좋아보인다. 추가로, 각 명령어들을 대문자로 많이 표현하는데, 실제로는 **대소문자를 구분하지 않는다**고 한다. 그럼에도 불구하고 대문자로 표현하는 이유를 추측하자면 SQL 명령어와 사용자 입력을 쉽게 구분하기 위함일 듯 하다. SQL의 컨벤션정도로 생각해서 지키는 것이 좋을 듯?

### 시작

DMBS 상위 랭크이면서 무료로 사용할 수 있는 MySQL을 선택했다. Installer를 따르다보면 중간에 어려운 내용들이 등장하긴 하는데 설치 방법을 잘 정리한 글이 엄청 많더라. 중요한 부분은 **[Accounts and Roles] 섹션에서 Root password를 꼭 기억해야 한다**는 점이다. 분실시 해결할 수 있는 방법이 있기는 한데.. 따라하기 만만치 않다. 

설치 후 MySQL 경로를 환경변수에 등록했다면 명령 프롬프트에 `mysql -uroot -p` 를 입력하여 관리자계정으로 로그인할 수 있다. 이 후 기억해둔 password를 입력하면 로그인 성공!

```powershell
# mysql -u root -p
Enter password: ****
```

![mysql_login.png](resource/MySQL/mysql_login.png)

그리고 예제로 사용할 DB도 필요하다. 몇 개월 전 아이폰14를 구매하면서 구글계정의 연락처 연동기능을 아주 잘 써먹었으니 phonebook DB를 만들어보도록 하겠다. 그리고 비교를 위한 dummy DB도!

```sql
CREATE DATABASE phonebook;
CREATE DATABASE dummy;
```

### 계정 생성과 권한

매 번 관리자계정을 사용하면 모양이 별로니까 `CREATE` 명령어로 새 사용자계정을 추가해두자.

```sql
# CREATE USER '{username}'@'localhost' IDENTIFIED BY '{password}';
CREATE USER 'authorA'@'localhost' IDENTIFIED BY '0000';

# CREATE USER '{username}'@'%' IDENTIFIED BY '{password}';
CREATE USER 'authorB'@'%' IDENTIFIED BY '0000';
```

두 명령어의 차이는 접속위치이다. authorA는 로컬에서만 접속할 수 있는 반면, authorB는 어디서나 접속할 수 있다. 물론 authorB는 정말 신뢰할 수 있는 사람이여야 한다. 만약 많은 권한을 갖고 있는 authorB의 계정과 DB의 주소가 노출되면 대참사가 벌어질지도..

`GRANT` 명령어를 사용하여 각 사용자계정마다 권한을 지정해줄 수 있다. authorA는 접속 위치가 특정되었으니 모든 권한을 부여하고, authorB는 위험하니 SELECT, INSERT만 부여해주자. `GRANT` 명령어를 사용하는 경우는 `FLUSH PRIVILEGES` 를 생략해도 된다.

```sql
# GRANT ALL PRIVILEGES ON {database}.* TO '{username}'@'localhost';
GRANT ALL PRIVILEGES ON phonebook.* TO 'authorA'@'localhost';
GRANT SELECT, INSERT ON phonebook.* TO 'authorB'@'%';
# FLUSH PRIVILEGES;
```

### 사용자계정으로 로그인

이제 관리자계정으로 할 일은 끝났으니 사용자계정으로 DB에 접근해보자. 이번에는 root password가 아닌, 조금 전에 사용자계정을 생성할 때 지정한 password를 입력해야 한다.

```powershell
# mysql -u authorA -p
Enter password: 0000
```

`SHOW` 명령어를 통해 authorA가 접근 가능한 DB목록을 살펴보면.. 기본 DB외에 phonebook이 있고 dummy는 없는 모씁!

```sql
SHOW databases;

# +--------------------+
# | Database           |
# +--------------------+
# | information_schema |
# | performance_schema |
# | phonebook          |
# +--------------------+
# 3 rows in set (0.00 sec)
```

### Table 생성하기

새 직장으로 이직하고 나서 팀 동료들의 연락처를 새로 등록해야 한다고 상상해보자. 연락처 추가를 위해 필수적인 내용은 **이름**과 **전화번호**이다. 하지만 현 사수와 전 사장님이 이름이 같다면 필히 구분해야 할 것이다. 그래서 우리는 직장, 나이, 성별 등의 추가 정보가 필요하기도 하다.

이를 바탕으로 table을 만든다고 하면… 사람을 구분해줄 수 있는 **member table**과 번호가 저장되어 있는 **digit table**을 만들면 될 듯 싶다..만! table 분할은 아직 시기상조이니 그냥 **mobile**이라는 table하나를 만들어보자.

`USE` 명령어로 phonebook DB를 선택하고 `CREATE` 명령어로 mobile table을 생성할 수 있다. 다만, authorB로 로그인했다면 아래와 같은 에러메세지를 마주하게 된다. 서술한대로, 이름과 번호는 필수column으로 하고 나머지는 부수적인 column으로 설정하자. 마지막에 `ENGINE`이라는 녀석은 table마다 지정할 수 있도록 되어있는데, MySQL에서는 INNODB를 default로 사용하고 있다. 나머지 ENGINE에 대해서는 따로 다뤄보려고 한다. TBU!

```sql
USE phonebook;

# authorA
CREATE TABLE mobile
    (
    _id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(32) NOT NULL,
    digit VARCHAR(11) NOT NULL,
		job VARCHAR(32),
		gender VARCHAR(32),
		age INT
    ) ENGINE=INNODB;

# authorB
ERROR 1142 (42000): CREATE command denied to user 'authorB'@'%' for table 'mobile’
```

![mobile_describe.png](resource/MySQL/mobile_describe.png)

### Data 다루기

포스트 초입에 **DML** 명령어들이 있다고 했다. 이제 DB와 table이 준비되었으니 이것들을 사용해서 data를 추가, 제거, 수정, 선택해보자. 아래는 권한이 있는 사용자의 경우 동작하는 SQL이다. 만약, authorB (권한이 `SELECT`, `INSERT` 밖에 없는 계정)이라면 `UPDATE`와 `DELETE` 명령어가 거부된다.

```sql
# authorA
USE phonebook;

INSERT INTO mobile values (1, 'memberA', '00000000000', '마법사', '남자', 30);
SELECT * FROM mobile;
# +-----+---------+-------------+--------+--------+------+
# | _id | name    | digit       | job    | gender | age  |
# +-----+---------+-------------+--------+--------+------+
# |   1 | memberA | 00000000000 | 마법사  | 남자   |   30 |
# +-----+---------+-------------+--------+--------+------+
# 1 row in set (0.00 sec)

UPDATE mobile SET job = '자택경비원', age = 35 where _id=1;
SELECT * FROM mobile;
# +-----+---------+-------------+------------+--------+------+
# | _id | name    | digit       | job        | gender | age  |
# +-----+---------+-------------+------------+--------+------+
# |   1 | memberA | 00000000000 | 자택경비원  | 남자   |   35 |
# +-----+---------+-------------+------------+--------+------+
# 1 row in set (0.00 sec)

DELETE FROM mobile where _id = 1;
SELECT * FROM mobile;
# Empty set (0.00 sec)
```
