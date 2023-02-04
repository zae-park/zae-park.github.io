---
title: MySQL로 연락처 DB 만들기
date: 2023-02-02 17:46:30
tags: [MySQL, pymysql]
categories: database
---

언젠가 휴대폰을 잃어버릴 경우를 염두에 두고 내 연락처를 backup 해두려고 한다. 이왕 backup하는 김에 mySQL로 DB를 만들어보자. 현재 휴대폰의 연락처를 살펴보면 요구사항은 대략 아래와 같다.

### 요구사항 명세표

- 연락처에는 여러 member가 등록될 수 있다.
- 각 member는 name, age, birth, gender, job, work을 key로 갖는다.
- 각 member는 여러개의 number를 가질 수도 있다.
- 각 number는 device, country를 key로 갖는다.
- 각 number는 하나의 member와 연결되어 있어야 한다.

## Database 만들기

지난 포스트에서는 CLI로 직접 database와 table을 만들었지만, 이번 포스트부터는 mySQL에서 제공하는 workbench를 사용하겠다. workbench의 설치 및 설정은 다른 글에서 잘 설명하고 있으니 참조하면 좋을 듯.

우선 database의 이름은 phonebook으로 정했고, 요구사항 명세표를 따라 두 table, members와 numbers를 아래와 같이 예제 data와 함께 추가한다. 

```markdown
### members

|_id|name|age|birth|gender|job|work|
|:---|---:|---:|---:|---:|---:|---:|
|1|박성재|30|1994-11-05|male|researcher|huinno|
|2|홍길동|98|1926-04-23|male|theif|korea|
|3|성재박|31|1993-10-02|male|researcher|huinno|
|4|재성박|30|1994-01-02|male|researcher|huinno|
|5|홍두깨|93|1931-02-11|male|homekeeper|home|
|6|박점례|90|1928-08-09|female|potato|home|

### numbers

|_id|digit|device|country|member_id|
|:---|---:|---:|---:|---:|
|1|010-2221-9944|mobile|korea|1|
|2|02-555-0899|home|korea|1|
|3|010-1234-5678|mobile|korea|3|
|4|010-1234-5679|mobile|korea|4|
|5|010-1234-5680|mobile|korea|5|
|6|010-1234-5681|mobile|korea|2|
|7|11-2334-15681|home|usa|6|
```

이제 아래의 query를 통해 두 table을 합칠 수 있다.

```sql
SELECT * FROM numbers INNER JOIN phonebook.members ON members.id = phonebook.numbers.member_id;
```

### Foreign key

추가로, 관계형 DB의 꽃이라 불리우는 foreign key라는 외래키를 추가해보자. 요구사항 명세표 중 마지막 항목이 외래키에 해당하는데, numbers의 각 row가 임의의 members의 row와 대응되는지 확인하는 용도이며, 동시에 members가 업데이트 되었을 때 일일히 numbers를 수정하지 않아도 되는 **참조 무결성**을 제공한다. 실무에서는 외래키의 사용 여부에 말이 많은데, 그래도 알아는 둬야 할 듯?

```sql
ALTER TABLE `phonebook`.`numbers` 
ADD CONSTRAINT `fk_member_id`
  FOREIGN KEY (`_id`)
  REFERENCES `phonebook`.`members` (`id`)
  ON DELETE CASCADE
  ON UPDATE CASCADE;
```

`ON DELETE`와 `ON UPDATE` 의 옵션으로 no action, set null, cascade, restrict이 있는데, 각각 reference가 수정 혹은 삭제되면 다음과 같은 동작을 한다.

- `set null` : NULL 값으로 변경
- `cascade` : 변경된 값을 따름
- `no action` or `restrict` : 변경 취소

### Python에서 database 참조하기

잘 만들어 둔 database는 잘 써야 맛이다. pymysql은 python에서 mysql을 참조할 수 있는 기능을 제공하는데 아주 꿀맛이다. 아래의 스크립트를 통해 위에서 생성한 두 table을 join해서 불러올 수 있다. 이 때 각 row는 tuple 형태로 제공된다. mySQL workbench와 동일한 table이 출력된다.

```python
with pymysql.connect(host="127.0.0.1", user={username}, password={password}, db='phonebook', charset='utf8') as conn:
		cursor = conn.cursor()
		sql = "SELECT * FROM numbers INNER JOIN phonebook.members ON members.id = phonebook.numbers.member_id"
		cursor.execute(sql)
		data = cursor.fetchall()
		print(data)

# (
# (1, '010-2221-9944', 'mobile', 'korea', 1, 1, '박성재', 30, '1994-11-05', 'male', 'researcher', 'huinno')
# (2, '02-555-0899', 'home', 'korea', 1, 1, '박성재', 30, '1994-11-05', 'male', 'researcher', 'huinno')
# ...
# (7, '11-2334-15681', 'home', 'usa', 6, 6, '박점례', 90, '1928-08-09', 'female', 'potato', 'home')
# )
```

![Untitled](resource/MySQL/workbench.png)

## 실습

간단한 준비는 끝났다. 이제 진짜 내 연락처를 DB에 백업해보자. 우선, 휴대폰에서 연락처를 export해야한다. 필자는 iPhone을 사용하고 있는데, **SA Contacts Lite**라는 앱을 통해 xls 포맷으로 연락처를 export하였다. 아마 gmail 계정에 연락처가 백업되어 있다면 더 쉽게도 가능할 것이다.

여튼 xls 혹은 csv 형태로 data가 저장되어 있다고 가정하자. 이 파일을 읽은 후 각 연락처를 mySQL에서 준비한 table에 맞게 가공한 후 `INSERT` query를 날리면 된다.

```python
with pymysql.connect(host="127.0.0.1", user='zae', password='9926', db='phonebook', charset='utf8') as conn:
    cursor = conn.cursor()

    member_sql = "INSERT INTO 'phonebook', 'members' ('name', 'age', 'birth', 'gender', 'job', 'work') VALUES (%s, %s, %s, %s, %s, %s)"
    number_sql = f"INSERT INTO 'phonebook', 'numbers' ('digit', 'device', 'country', 'member_id') " \
                 f"VALUES (%s, %s, %s, %s)"

    for i in range(len(my_phonebook)):
        data = my_phonebook.iloc[i, :]
        name, birth, job, work = data['Firstname'], data['Birthday'], data['Jobtitle'], data['Company']
        cursor.execute(member_sql, (name, '0', str(birth), 'NULL', str(job), str(work)))

        number = data['Phonenumbers']
        device, digit = number.split(': ')
        cursor.execute(number_sql, (digit, device, 'korea', str(i + 1)))
```
