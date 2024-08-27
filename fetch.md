
---

# **JPA 페치 계획, 페치 전략, 페치 프로파일**

---

## **목차**

1. 페치 계획 (Fetch Plan)
   - 정의
   - 중요성
   - 페치 타입
     - 지연 로딩 (FetchType.LAZY)
     - 즉시 로딩 (FetchType.EAGER)
   - 지연 로딩과 즉시 로딩의 비교
   - 페치 계획 설정 방법
2. 페치 전략 (Fetch Strategy)
   - 기본 개념
   - N+1 문제
   - 데카르트 곱 문제
   - 일괄 데이터 프리페치 (PreFetch)
   - 다양한 페치 전략
     - FetchMode.JOIN
     - FetchMode.SELECT
     - FetchMode.SUBSELECT
   - 페치 전략 설정 방법
3. 페치 프로파일 (Fetch Profile)
   - 기본 개념
   - 엔티티 그래프 (Entity Graph)
     - 기본 엔티티 그래프
     - 복잡한 엔티티 그래프
   - 엔티티 그래프 활용 방법
   - 동적 페치 계획
4. 결론 및 Q&A

---

## **1. 페치 계획 (Fetch Plan)**

### **1.1 정의**

페치 계획은 JPA에서 객체를 로드할 때 어떤 데이터를 데이터베이스에서 가져올지를 정의하는 전략입니다. 이는 성능 최적화와 메모리 사용을 고려하여 효율적인 데이터 액세스를 지원합니다.

### **1.2 중요성**

페치 계획은 데이터베이스 호출을 최소화하고, 성능을 향상시키며, 메모리 사용을 절감하는 데 중요합니다. 이를 통해 데이터 로딩과 관련된 성능 문제를 예방할 수 있습니다.

### **1.3 페치 타입**

JPA에서는 두 가지 주요 페치 타입을 지원합니다: `FetchType.LAZY`와 `FetchType.EAGER`.

#### **1.3.1 지연 로딩 (FetchType.LAZY)**

- **설명**: `FetchType.LAZY`는 연관된 엔티티나 컬렉션을 실제로 사용할 때까지 데이터베이스에서 로드하지 않습니다. 이 방식은 성능과 메모리 사용 측면에서 유리할 수 있지만, 객체를 실제로 사용할 때 추가적인 데이터베이스 호출이 발생할 수 있습니다.

- **특징**:
  - **프록시 객체**: 하이버네이트는 `Proxy` 객체를 사용하여 실제 객체를 대체합니다. 이 프록시는 데이터베이스에서 실제 객체를 가져오는 역할을 합니다.
  - **컬렉션 래퍼**: `PersistentSet`과 같은 컬렉션 래퍼를 사용하여 지연 로딩을 구현합니다.

**예시 코드:**
```java
@Entity
public class Item {
    @OneToMany(mappedBy = "item", fetch = FetchType.LAZY)
    private Set<Bid> bids = new HashSet<>();
}
```

**프록시 사용 예시:**
```java
Item item = em.find(Item.class, ITEM_ID);
System.out.println(item.getId()); // SQL SELECT 발생하지 않음
System.out.println(item.getBids().size()); // SQL SELECT 발생
```

#### **1.3.2 즉시 로딩 (FetchType.EAGER)**

- **설명**: `FetchType.EAGER`는 엔티티를 조회할 때 연관된 모든 데이터를 즉시 로드합니다. 이는 데이터에 즉시 접근할 수 있도록 하지만, 성능과 메모리 사용 측면에서 불리할 수 있습니다.

- **특징**:
  - **JOIN FETCH**: 관련된 엔티티를 로드할 때 SQL JOIN을 사용하여 한 번의 쿼리로 모든 데이터를 가져옵니다.

**예시 코드:**
```java
@Entity
public class Item {
    @ManyToOne(fetch = FetchType.EAGER)
    private User seller;
}
```

**Test 예시:**
```java
Item item = em.find(Item.class, ITEM_ID);
// SQL JOIN 쿼리 발생: select i.*, u.* from ITEM i left join USERS u on u.ID = i.SELLER_ID
```

### **1.4 지연 로딩과 즉시 로딩의 비교**

- **지연 로딩의 장점**:
  - 성능 최적화: 필요한 데이터만 로드하여 쿼리 수를 줄입니다.
  - 메모리 사용 감소: 객체를 필요할 때만 로드하여 메모리 사용을 줄입니다.

- **지연 로딩의 단점**:
  - N+1 문제: 지연 로딩을 사용할 때 연관된 데이터를 접근할 때 추가 쿼리가 발생할 수 있습니다.
  - 지연 로딩된 엔티티가 프록시 객체로 제공되기 때문에, 프록시를 사용하는 동안 데이터베이스에 추가 호출이 발생할 수 있습니다.

- **즉시 로딩의 장점**:
  - 데이터 접근 용이: 데이터가 즉시 로드되므로 관련 데이터를 한 번에 로드할 수 있습니다.
  - 코드 단순화: 연관된 엔티티를 사용하는 코드가 간단해질 수 있습니다.

- **즉시 로딩의 단점**:
  - 성능 저하: 불필요한 데이터까지 로드되어 성능이 저하될 수 있습니다.
  - 메모리 사용 증가: 모든 연관 데이터를 한 번에 로드하여 메모리 사용이 증가할 수 있습니다.

### **1.5 페치 계획 설정 방법**

페치 계획을 설정하려면 엔티티의 `@Fetch` 어노테이션을 사용하거나 JPQL 쿼리에서 `JOIN FETCH`를 사용할 수 있습니다.

**JPQL 예시:**
```java
List<Item> items = em.createQuery("select i from Item i join fetch i.seller", Item.class).getResultList();
```

**하이버네이트 설정 예시:**
```java
@Entity
public class Item {
    @ManyToOne(fetch = FetchType.LAZY)
    private User seller;
}
```

---

## **2. 페치 전략 (Fetch Strategy)**

### **2.1 기본 개념**

페치 전략은 데이터베이스에서 연관된 데이터를 어떻게 로드할지를 정의하는 방법입니다. 올바른 페치 전략을 선택하는 것은 데이터베이스 성능을 최적화하고, 쿼리 효율성을 향상시키는 데 중요합니다.

### **2.2 N+1 문제**

- **문제 설명**: 기본 페치 계획을 지연 로딩으로 설정했을 때 발생합니다. 하나의 쿼리로 엔티티를 조회한 후, 각 엔티티의 연관된 데이터를 로드하기 위해 N개의 추가 쿼리가 발생합니다. 이로 인해 성능이 크게 저하됩니다.

**예시 코드 (N+1 문제 발생):**
```java
List<Item> items = em.createQuery("select i from Item i", Item.class).getResultList();
// 1 + N 쿼리 발생
for (Item item : items) {
    System.out.println(item.getSeller().getUsername()); // 추가 SELECT 발생
}
```

**해결 방법**:
- **즉시 로딩**: 연관된 데이터를 한 번의 쿼리로 로드하여 N+1 문제를 방지합니다.
- **JOIN FETCH**: JPQL 쿼리에서 `JOIN FETCH`를 사용하여 연관된 데이터를 한 번에 로드합니다.

**예시 코드 (해결 방법):**
```java
List<Item> items = em.createQuery("select i from Item i join fetch i.seller", Item.class).getResultList();
// SQL JOIN 쿼리 발생: select i.*, s.* from ITEM i left join USER s on s.ID = i.SELLER_ID
```

### **2.3 데카르트 곱 문제**

- **문제 설명**: `FetchType.EAGER`로 설정된 `@ManyToMany` 관계에서 JOIN을 사용할 경우, 결과가 중복되어 비효율적인 데이터가 반환됩니다. 이는 결과적으로 성능 저하를 초래할 수 있습니다.

**예시 코드 (데카르트 곱 문제 발생):**
```java
@Entity
public class Item {
    @OneToMany(mappedBy = "item", fetch = FetchType.EAGER)
    private Set<Bid> bids = new HashSet<>();
    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "IMAGE")
    @Column(name = "FILENAME")
    private Set<String> images = new HashSet<>();
}
```

**해결 방법**:
- **지연 로딩 사용**: 필요한 시점에만 데이터를 로드하여 성능을 최적화합니다.
- **일괄 페치**: `@BatchSize`를 사용하여 N개의 쿼리 대신 일괄적으로 데이터를 로드합니다.

**예시 코드 (지연 로딩 사용):**
```java
@Entity
public class Item {
    @OneToMany(mappedBy = "item", fetch = FetchType.LAZY)
    private Set<Bid> bids = new HashSet<>();
}
```

**예시 코드 (일괄 페치 사용):**
```java
@Entity


@BatchSize(size = 10)
public class User {
    // ...
}
```

### **2.4 일괄 데이터 프리페치 (PreFetch)**

- **설명**: 일괄 데이터 프리페치는 지연 로딩을 사용하면서도 성능을 최적화할 수 있는 방법입니다. N개의 쿼리 대신 일괄적으로 데이터를 로드하여 성능을 개선합니다.

#### **2.4.1 FetchMode.JOIN**

- **설명**: `FetchMode.JOIN`은 SQL JOIN을 사용하여 연관된 엔티티를 한 번의 쿼리로 로드합니다.

**예시 코드:**
```java
@Entity
public class Item {
    @ManyToOne(fetch = FetchType.LAZY)
    @Fetch(FetchMode.JOIN)
    private User seller;
}
```

**Test 예시:**
```java
Item item = em.find(Item.class, ITEM_ID);
// JOIN 쿼리 발생
```

#### **2.4.2 FetchMode.SELECT**

- **설명**: `FetchMode.SELECT`는 연관된 데이터를 각각의 SELECT 쿼리로 로드합니다. 성능은 다소 떨어질 수 있지만, 간단한 로딩 시나리오에서 유용할 수 있습니다.

**예시 코드:**
```java
@Entity
public class Item {
    @ManyToOne(fetch = FetchType.EAGER)
    @Fetch(FetchMode.SELECT)
    private User seller;
}
```

**Test 예시:**
```java
Item item = em.find(Item.class, ITEM_ID);
// 여러 개의 SELECT 쿼리 발생
```

#### **2.4.3 FetchMode.SUBSELECT**

- **설명**: `FetchMode.SUBSELECT`는 서브쿼리를 사용하여 컬렉션을 로드합니다. 이는 N+1 문제를 완화하는 데 유용합니다.

**예시 코드:**
```java
@Entity
public class Item {
    @OneToMany(mappedBy = "item")
    @Fetch(FetchMode.SUBSELECT)
    private Set<Bid> bids = new HashSet<>();
}
```

**Test 예시:**
```java
List<Item> items = em.createQuery("select i from Item i", Item.class).getResultList();
// 서브쿼리 사용
```

### **2.5 페치 전략 설정 방법**

페치 전략을 설정하려면 엔티티의 `@Fetch` 어노테이션을 사용하거나 JPQL 쿼리에서 `JOIN FETCH`를 사용할 수 있습니다.

**JPQL 예시:**
```java
List<Item> items = em.createQuery("select i from Item i join fetch i.seller", Item.class).getResultList();
```

**하이버네이트 설정 예시:**
```java
@Entity
public class Item {
    @ManyToOne(fetch = FetchType.LAZY)
    @Fetch(FetchMode.JOIN)
    private User seller;
}
```

---

## **3. 페치 프로파일 (Fetch Profile)**

### **3.1 엔티티 그래프 (Entity Graph)**

- **설명**: 엔티티 그래프는 페치 계획을 동적으로 재정의하거나 보강하여 필요한 데이터를 효율적으로 로드하는 기능입니다. 이를 통해 쿼리 성능을 개선하고 데이터 로딩을 최적화할 수 있습니다.

#### **3.1.1 기본 엔티티 그래프**

- **설명**: 기본 엔티티 그래프를 정의하여 연관된 엔티티를 미리 로드할 수 있습니다.

**예시 코드:**
```java
@NamedEntityGraph(name = "ItemSeller", attributeNodes = {
    @NamedAttributeNode("seller")
})
@Entity
public class Item {
    @ManyToOne(fetch = FetchType.LAZY)
    private User seller;
}
```

**Test 예시:**
```java
Map<String, Object> properties = new HashMap<>();
properties.put("jakarta.persistence.loadgraph", em.getEntityGraph("ItemSeller"));

Item item = em.find(Item.class, ITEM_ID, properties);
// JOIN 쿼리 발생
```

#### **3.1.2 복잡한 엔티티 그래프**

- **설명**: 복잡한 엔티티 그래프를 정의하여 여러 연관 엔티티를 로드할 수 있습니다.

**예시 코드:**
```java
@NamedEntityGraphs({
    @NamedEntityGraph(name = "BidBidderItemSellerBids",
        attributeNodes = {
            @NamedAttributeNode(value = "bidder"),
            @NamedAttributeNode(value = "item", subgraph = "ItemSellerBids")
        },
        subgraphs = {
            @NamedSubgraph(name = "ItemSellerBids",
                attributeNodes = {
                    @NamedAttributeNode("seller"),
                    @NamedAttributeNode("bids")
                })
        }
    )
})
@Entity
public class Bid {
    @Id
    private Long id;
    @ManyToOne(fetch = FetchType.LAZY)
    private User bidder;
    @ManyToOne(fetch = FetchType.LAZY)
    private Item item;
}
```

**Test 예시:**
```java
Map<String, Object> properties = new HashMap<>();
properties.put("jakarta.persistence.loadgraph", em.getEntityGraph("BidBidderItemSellerBids"));

Bid bid = em.find(Bid.class, BID_ID, properties);
// 복잡한 JOIN 쿼리 발생
```

### **3.2 엔티티 그래프 활용 심화**

- **설명**: 복잡한 엔티티 그래프를 정의하여 다양한 연관관계를 한 번에 로드할 수 있습니다. 이를 통해 성능을 최적화하고 데이터 로딩을 효율적으로 관리할 수 있습니다.

**예시 코드:**
```java
@NamedEntityGraphs({
    @NamedEntityGraph(name = "ComplexGraph",
        attributeNodes = {
            @NamedAttributeNode(value = "seller"),
            @NamedAttributeNode(value = "bids", subgraph = "BidDetails")
        },
        subgraphs = {
            @NamedSubgraph(name = "BidDetails",
                attributeNodes = {
                    @NamedAttributeNode("bidder"),
                    @NamedAttributeNode("item")
                })
        }
    )
})
@Entity
public class Item {
    @Id
    private Long id;
    @ManyToOne(fetch = FetchType.LAZY)
    private User seller;
    @OneToMany(mappedBy = "item", fetch = FetchType.LAZY)
    private Set<Bid> bids = new HashSet<>();
}
```

**Test 예시:**
```java
Map<String, Object> properties = new HashMap<>();
properties.put("jakarta.persistence.loadgraph", em.getEntityGraph("ComplexGraph"));

Item item = em.find(Item.class, ITEM_ID, properties);
// 복잡한 JOIN 쿼리 발생
```

### **3.3 동적 페치 계획**

- **설명**: 동적 페치 계획을 사용하여 애플리케이션의 실행 시간에 따라 적절한 페치 계획을 설정할 수 있습니다. 이는 복잡한 데이터 모델에서 유용할 수 있습니다.

**예시 코드:**
```java
@Entity
public class Item {
    @ManyToOne(fetch = FetchType.LAZY)
    private User seller;
    @OneToMany(mappedBy = "item", fetch = FetchType.LAZY)
    private Set<Bid> bids = new HashSet<>();
}
```

**JPQL 예시:**
```java
List<Item> items = em.createQuery("select i from Item i join fetch i.seller join fetch i.bids", Item.class).getResultList();
```

---

## **결론**

페치 계획, 페치 전략, 페치 프로파일은 JPA에서 데이터베이스 성능을 최적화하고 효율적으로 데이터를 로드하기 위한 핵심 요소입니다. 적절한 페치 전략과 프로파일을 설정하여 데이터 액세스 패턴을 최적화하고, 다양한 사용 사례에 맞는 효과적인 데이터 로딩을 구현할 수 있습니다. 발표를 통해 JPA의 페치 계획과 전략을 잘 이해하고, 실제 애플리케이션에서 성능을 최적화하는 데 도움이 되기를 바랍니다.
