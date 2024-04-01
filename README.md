# 인프런 강의

해당 저장소의 `README.md`는 인프런 김영한님의 SpringBoot 강의 시리즈를 듣고 Spring 프레임워크의 방대한 기술들을 복기하고자 공부한 내용을 가볍게 정리한 것입니다. 문제가 될 시 삭제하겠습니다.



## 해당 프로젝트에서 배우는 내용

* 섹션 1 | 데이터 접근 기술 - 시작



# 섹션 1 | 데이터 접근 기술 - 시작

## 데이터 접근 기술 소개

실무에서 주로 사용하는 다음과 같은 다양한 데이터 접근 기술들을 학습하게 된다.



**SQL Mapper 주요 기능**

* 개발자는 SQL만 작성하면 해당 SQL의 결과를 개겣로 편리하게 매핑해준다.
* JDBC를 직접 사용할 때 발생하는 여러가지 중복을 제거해주고, 기타 개발자에게 여러가지 편리한 기능을 제공한다.
* 예: JdbcTemplate, MyBatis



**ORM 주요 기능**

* JdbcTemplate이나 MyBatis 같은 SQL 매퍼 기술은 SQL을 개발자가 직접 작성해야 하지만, JPA를 사용하면 기본적인 SQL은 JPA가 대신 작성하고 처리해준다. 
  * 개발자는 저장하고 싶은 객체를 마치 자바 컬렉션에 저장하고 조회하듯이 사용하면 ORM 기술이 데이터베이스에 해당 객체를 저장하고 조회해준다.
* 스프링 데이터 JPA, Querydsl은 JPA를 더 편리하게 사용할 수 있게 도와주는 프로젝트들이다.
* 예: JPA, Hibernate, 스프링 데이터 JPA, Querydsl



## 프로젝트 구조 설명1 - 기본

> 학습에서 나온 내용을 모두 요약하기 보다는 새로 배우는 내용에 대해서만 정리를 하려고 한다.



[ItemUpdateDto]

```java
package hello.itemservice.repository;

import lombok.Data;

@Data
public class ItemUpdateDto {
    private String itemName;
    private Integer price;
    private Integer quantity;

    public ItemUpdateDto() {
    }

    public ItemUpdateDto(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```



#### Dto(data transfer object)

* 데이터 전송 객체
* DTO는 기능은 없고 데이터 전달을 하는 용도로 사용되는 객체를 뜻한다.
* DTO에 기능이 없으면 안되는 것은 아니지만, 객체의 주 목적이 데이터를 전송하는 것이라면 DTO라고 할 수 있다.
  * 또한 DTO를 뒤에 붙여준다면 용도를 알 수 있다는 장점이 있다.







## 프로젝트 구조 설명3 - 테스트

#### 테스트 코드

[ItemRepositoryTest]

* `afterEach()`
  * 인터페이스에는 `clearStore()`가 없기 때문에 `MemoryItemRepository`인 경우에만 다운 캐스팅을 해서 데이터를 초기화한다.

```java
package hello.itemservice.domain;

import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import hello.itemservice.repository.memory.MemoryItemRepository;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@Slf4j
@SpringBootTest
class ItemRepositoryTest {

    @Autowired
    ItemRepository itemRepository;

    @Autowired
    ApplicationContext applicationContext;


    /**
     * 테스트는 서로 영향을 주면 안되므로, 각각의 테스트가 끝나고 나면 저장한 데이터를 제거해야 한다.
     * 인터페이스에는 clearStore()가 없기 때문에 MemoryItemRepository인 경우에만 다운 캐스팅을 해서 데이터를 초기화 한다.
     */
    @AfterEach
    void afterEach() {
        //MemoryItemRepository 의 경우 제한적으로 사용
        if (itemRepository instanceof MemoryItemRepository) {
            ((MemoryItemRepository) itemRepository).clearStore();
        }
    }

    @Test
    void applicationContext() {
        String[] beanDefinitionNames = applicationContext.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            log.info("beanDefinitionName = {}", beanDefinitionName);
        }
    }

    @Test
    void save() {
        //given
        Item item = new Item("itemA", 10000, 10);

        //when
        Item savedItem = itemRepository.save(item);

        //then
        Item findItem = itemRepository.findById(item.getId()).get();
        assertThat(findItem).isEqualTo(savedItem);
    }

    @Test
    void updateItem() {
        //given
        Item item = new Item("item1", 10000, 10);
        Item savedItem = itemRepository.save(item);
        Long itemId = savedItem.getId();

        //when
        ItemUpdateDto updateParam = new ItemUpdateDto("item2", 20000, 30);
        itemRepository.update(itemId, updateParam);

        //then
        Item findItem = itemRepository.findById(itemId).get();
        assertThat(findItem.getItemName()).isEqualTo(updateParam.getItemName());
        assertThat(findItem.getPrice()).isEqualTo(updateParam.getPrice());
        assertThat(findItem.getQuantity()).isEqualTo(updateParam.getQuantity());
    }

    @Test
    void findItems() {
        //given
        Item item1 = new Item("itemA-1", 10000, 10);
        Item item2 = new Item("itemA-2", 20000, 20);
        Item item3 = new Item("itemB-1", 30000, 30);

        itemRepository.save(item1);
        itemRepository.save(item2);
        itemRepository.save(item3);

        //둘 다 없음 검증
        test(null, null, item1, item2, item3);
        test("", null, item1, item2, item3);

        //itemName 검증
        test("itemA", null, item1, item2);
        test("temA", null, item1, item2);
        test("itemB", null, item3);

        //maxPrice 검증
        test(null, 10000, item1);

        //둘 다 있음 검증
        test("itemA", 10000, item1);
    }

    void test(String itemName, Integer maxPrice, Item... items) {
        List<Item> result = itemRepository.findAll(new ItemSearchCond(itemName, maxPrice));
        assertThat(result).containsExactly(items); // 해당 아이템들의 포함 여부 뿐만 아니라 순서까지 체크한다.
    }
}
```



#### 인터페이스를 테스트하자

여기서는 `MemoryItemRepository` 구현체를 테스트 하는 것이 아니라 `ItemRepository` 인터페이스를 테스트하는 것을 확인할 수 있다. <u>인터페이스를 대상으로 테스트하면 향후 다른 구현체로 변경되었을 때 해당 구현체가 잘 동작하는지 같은 테스트로 편리하게 검증할 수 있다.</u>



## 데이터베이스 테이블 생성

이제부터 다양한 데이터 접근 기술을 활용해 메모리가 아닌 데이터베이스에 데이터를 보관하는 방법을 알아보자



실습을 위해 H2 데이터베이스에 `item` 테이블을 생성과 데이터를 넣자

* `generated by default as identity`
  * `identity` 전략이라고 하는데, 기본 키 생성을 데이터베이스에 위임하는 방법이다. 
  * MySQL의 Auto Increment와 같은 방법이다.

```sql
 drop table if exists item CASCADE;
 create table item
 (
     id        bigint generated by default as identity,
     item_name varchar(10),
     price     integer,
     quantity  integer,
     primary key (id)
 );
 
 insert into item(item_name, price, quantity) values ('ItemTest', 10000, 10);
```



#### 참고 - 권장하는 식별자 선택 전략

**데이터베이스의 기본 키는 다음 3가지 조건을 모두 만족해야 한다.**

1. `null` 값은 허용하지 않는다.
2. 유일해야 한다.
3. 변해선 안 된다.



**테이블의 기본 키를 선택하는 전략은 크게 2가지가 있다.**

* 자연 키
  * 비즈니스에 의미가 있는 키
  * 예: 주민등록번호, 이메일, 전화번호
* 대리 키
  * 비즈니스와 관련 없는 임의로 만들어진 키, 대체 키로도 불린다.
  * 예: 오라클 시퀀스, auto_increment, identity, 키 생성 테이블 사용



**현실과 비즈니스 규칙은 생각보다 자주 변경되기 때문에 현재와 미래에도 변경이 없을 자연 키 보다는 대리 키를 권장한다.**