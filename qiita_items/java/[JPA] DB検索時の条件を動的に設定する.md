Spring Boot と JPA を使用する環境で、動的に条件を設定する方法についてのメモ。
本記事では次の条件を指定するケースに触れる。

* 本記事で触れる条件式
 * 指定した値と等しい
 * in句の指定
 * LEFT OUTER JOIN

# 構成
次の構成とする。

* Spring Boot の構成
 * [SPRING INITIALIZR](https://start.spring.io/) でプロジェクトを生成している
 * その際 Search for dependencies には JPA, H2, Lombok を指定している
* DB のテーブル構成
 * 親テーブルとして Company テーブルを定義する
 * 子テーブルとして Customer テーブルを定義する
 * 両者は company_id を外部キー(　FOREIGN KEY　) として親子関係にある

# 動的な条件の生成
* 動的に条件を設定したいテーブルの Repository インターフェースで JpaSpecificationExecutor を継承する
* クエリメソッドを作成する。具体的には
 * Specification の匿名クラスを作り
 * その中で toPredicate メソッドをオーバーライドする

実際のコードは次の通り。

## JpaSpecificationExecutor の継承した Repository インターフェースを定義する

```java:Repository
package com.example.repository;

import com.example.domain.Company;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;

public interface CompanyRepository extends JpaRepository<Company, Integer>, JpaSpecificationExecutor<Company> {
}
```

# クエリメソッドを定義する
ここでは前掲の3つの条件に対応したメソッドを定義する。

## 指定した値と等しい

```java:「引数のcompanyNameと等しいとなる」条件式を生成する
private Specification<Company> companyNameEqual(String companyName) {
    return StringUtils.isEmpty(companyName) ? null : new Specification<Company>() {
        @Override
        public Predicate toPredicate(Root<Company> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
            return criteriaBuilder.equal(root.get("companyName"), companyName);
        }
    };
}
```

## in句の指定

```java:「引数のcompanyNameListをin句とする」条件式を生成する
private Specification<Company> companyNameInclude(List<String> companyNameList) {
    return companyNameList.size() == 0 ? null : new Specification<Company>() {
        @Override
        public Predicate toPredicate(Root<Company> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
            return root.get("companyName").in(companyNameList);
        }
    };
}
```

## LEFT OUTER JOIN

```java:「親テーブルと子テーブルを左側外部結合し、かつ引数のcustomerCodeと等しいとなる」条件式を生成する
private Specification<Company> customerCodeEqual(String customerCode) {
    return StringUtils.isEmpty(customerCode) ? null : new Specification<Company>() {
        @Override
        public Predicate toPredicate(Root<Company> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
            return criteriaBuilder.equal(root.join("customers", JoinType.LEFT).get("customerCode"), customerCode);
        }
    };
}
```

# DBアクセス
上記のクエリメソッドを使用してDBアクセスを行う。

```java:クエリメソッドを使用する
private execute() {
    String companyName = "hogeCompany";

    List<String> companyNameList = new ArrayList<>();
    companyNameList.add("hogeCompany");
    companyNameList.add("piyoCompany");

    String customerCode = "hogehoge";

    List<Company> companies = this.companyRepository.findAll(Specifications
            .where(this.companyNameEqual(companyName))
            .and(this.companyNameInclude(companyNameList))
            .and(this.customerCodeEqual(customerCode))
    );  
}
```

# クエリメソッドとそれを利用する際のポイント
上記までのコードにおけるポイントは次の通り。

* Repository インターフェースは **JpaSpecificationExecutor** を **継承するだけ** で良い
* 各クエリメソッドは引数に設定されたデータが 「空文字("")」 や リストサイズが「0」の場合に **null** を返す
* その場合 **null を返却したクエリメソッドの条件式は生成されない**

また条件式を追加したい場合はクエリメソッドを作成し 

```java:条件式を追加したい場合
    .where(this.companyNameEqual(companyName))
    .and(this.companyNameInclude(companyNameList))
    .and(this.customerCodeEqual(customerCode))
    // ここに次のような感じで追加したクエリメソッドを設定する
    .and(this.nameContains(customerName))
```

と同じ要領で追加してやれば良い。

# 実行時に生成されたクエリをみる
実際にコードを実行した際に生成されるクエリは以下のとおり。

## 条件をすべて有効にしたときのクエリ

```sql:条件をすべて有効にしたときのクエリ
select
  company0_.company_id as company_1_0_,
  company0_.company_name as company_2_0_ 
from
  company company0_ 
left outer join
  customers customers1_ 
on
  company0_.company_id = customers1_.company_id 
where
  company0_.company_name = 'hogeCompany' 
and (
  company0_.company_name in ('hogeCompany', 'piyoCompany')
) 
and
  customers1_.customer_code = 'hogehoge'
```

## companyName を条件に指定したときのクエリ

```sql:companyNameを条件に指定したときのクエリ
/*
 * companyName を条件に指定したときのクエリ
 * つまり他の条件は
 * companyNameList.size() == 0 か StringUtils.isEmpty(customerCode) == true
 * の場合
 */
select
  company0_.company_id as company_1_0_,
  company0_.company_name as company_2_0_ 
from
  company company0_ 
where
  company0_.company_name = 'hogeCompany'
```

## companyNameList を条件に指定したときのクエリ

```sql:companyNameListを条件に指定したときのクエリ
/*
 * companyNameList を条件に指定したときのクエリ
 * つまり他の条件は
 * StringUtils.isEmpty(companyName) == true か StringUtils.isEmpty(customerCode) == true
 * の場合
 */
select
  company0_.company_id as company_1_0_,
  company0_.company_name as company_2_0_ 
from
  company company0_ 
where
  company0_.company_name in ('hogeCompany', 'piyoCompany')
```

## customerCode を条件に指定したときのクエリ

```sql:customerCodeを条件に指定したときのクエリ
/*
 * customerCode を条件に指定したときのクエリ
 * つまり他の条件は
 * StringUtils.isEmpty(companyName) == true か companyNameList.size() == 0
 * の場合
 */
select
  company0_.company_id as company_1_0_,
  company0_.company_name as company_2_0_ 
from
  company company0_ 
left outer join
  customers customers1_ 
on
  company0_.company_id = customers1_.company_id 
where
  customers1_.customer_code = 'hogehoge'
```

## 条件をなにも指定しなかったときのクエリ

```sql:条件をなにも指定しなかったときのクエリ
select
  company0_.company_id as company_1_0_,
  company0_.company_name as company_2_0_ 
from
  company company0_
```

# 最後に
本記事を書く際に作成した各ファイルのコードを忘れないように載せておく。

## テーブル定義とデータ

```sql:schema.sql
CREATE TABLE IF NOT EXISTS company(
    company_id INT PRIMARY KEY AUTO_INCREMENT,
    company_name VARCHAR(30)
);

CREATE TABLE IF NOT EXISTS customer
(
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_code VARCHAR(30),
    company_id INT,
    first_name VARCHAR(30),
    last_name VARCHAR(30),
    FOREIGN KEY(company_id) REFERENCES company(company_id)
);
```

```sql:data.sql
INSERT INTO company(company_name) VALUES('hogeCompany');
INSERT INTO company(company_name) VALUES('piyoCompany');

INSERT INTO customer(customer_code, company_id, first_name, last_name) VALUES('hogehoge', 1, 'hoge', 'hoge');
INSERT INTO customer(customer_code, company_id, first_name, last_name) VALUES('barbar', 1, 'bar', 'bar');
INSERT INTO customer(customer_code, company_id, first_name, last_name) VALUES('booboo', 1, 'boo', 'boo');
INSERT INTO customer(customer_code, company_id, first_name, last_name) VALUES('piyopiyo', 2, 'piyo', 'piyo');
```

## 各テーブルにおける Entity と Repository
上記構成におこえる Entity クラスと Repository クラスは次の通り。

```java:Entity(Company.java)
package com.example.domain;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.*;
import java.util.List;

@Entity
@Table(name = "company")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Company {

    @Id
    @GeneratedValue
    @Column(name = "company_id")
    private Integer companyId;

    @Column(name = "company_name", nullable = false)
    private String companyName;

    @OneToMany
    @JoinColumn(name = "company_id", referencedColumnName = "company_id")
    private List<Customer> customers;
}
```

```java:Entity(Customer.java)
package com.example.domain;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Entity
@Table(name = "customers")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Customer {

    @Id
    @GeneratedValue
    @Column(name = "customer_id")
    private Integer customerId;

    @Column(name = "customer_code", nullable = false)
    private String customerCode;

    @Column(name ="company_id", nullable = false)
    private Integer companyId;

    @Column(name = "first_name", nullable = false)
    private String firstName;

    @Column(name = "last_name", nullable = false)
    private String lastName;
}
```

```java:Repository(CompanyRepository.java)
package com.example.repository;

import com.example.domain.Company;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;

public interface CompanyRepository extends JpaRepository<Company, Integer>, JpaSpecificationExecutor<Company> {
}
```

```java:Repository(Customer.java)
package com.example.repository;

import com.example.domain.Customer;
import org.springframework.data.jpa.repository.JpaRepository;

public interface CustomerRepository extends JpaRepository<Customer, Integer>{
}
```

## 実行クラス

```java:JpaApplication.java
package com.example;

import com.example.domain.Company;
import com.example.repository.CompanyRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.data.jpa.domain.Specifications;
import org.springframework.util.StringUtils;

import javax.persistence.criteria.*;
import java.util.ArrayList;
import java.util.List;

@SpringBootApplication
public class JpaApplication implements CommandLineRunner {

    @Autowired
    CompanyRepository companyRepository;

    private Specification<Company> companyNameEqual(String companyName) {
        return StringUtils.isEmpty(companyName) ? null : new Specification<Company>() {
            @Override
            public Predicate toPredicate(Root<Company> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
                return criteriaBuilder.equal(root.get("companyName"), companyName);
            }
        };
    }

    private Specification<Company> companyNameInclude(List<String> companyNameList) {
        return companyNameList.size() == 0 ? null : new Specification<Company>() {
            @Override
            public Predicate toPredicate(Root<Company> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
                return root.get("companyName").in(companyNameList);
            }
        };
    }

    private Specification<Company> customerCodeEqual(String customerCode) {
        return StringUtils.isEmpty(customerCode) ? null : new Specification<Company>() {
            @Override
            public Predicate toPredicate(Root<Company> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
                return criteriaBuilder.equal(root.join("customers", JoinType.LEFT).get("customerCode"), customerCode);
            }
        };
    }

    @Override
    public void run(String... strings) throws Exception {
        String companyName = "hogeCompany";

        List<String> companyNameList = new ArrayList<>();
        companyNameList.add("hogeCompany");
        companyNameList.add("piyoCompany");

        String customerCode = "hogehoge";

        List<Company> companies = this.companyRepository.findAll(Specifications
                .where(this.companyNameEqual(companyName))
                .and(this.companyNameInclude(companyNameList))
                .and(this.customerCodeEqual(customerCode))
        );
    }

    public static void main(String[] args) {
        SpringApplication.run(JpaApplication.class, args);
    }
}
```
