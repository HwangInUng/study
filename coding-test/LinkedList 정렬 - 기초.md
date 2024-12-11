## LinkeddList 정렬 - 기초
>Leetcode의 21.Merge Two Sroted Lists를 풀어보면서 부족한 부분이 있어서 다시 정리하고 넘어가려한다.

### 문제 설명
문제에서 최종적으로 요구하는 내용은 이미 오름차순으로 정렬된 2개의 LinkedList를 순서대로 병합하여 반환하는 것이며,
다음과 같은 고려사항이 주어진다.

- ListNode 타입의 파라미터 2개가 전달됨
- 2개의 Node를 하나의 정렬된 리스트로 구성
- Head 노드를 return

#### 부족한 부분 식별
여기서 내가 실수한 부분은 `리스트로 구성하여 반환하라`는 문구만 보고, `왜 리스트 형태가 아닌데 리스트로 구성해서 반환해야하지?`라는
생각을 했었다. 이 부분은 누가 보더라도 링크드 리스트에 대한 이해가 부족했다고 판단할 수 있다.

그래서 시간이 좀 소요되었지만 차분히 문제를 읽다가 막히는 부분은 지식이 부족하다고 인정하고, 이론적 내용을 참고하면서 해결해보았다.

먼저 파라미터로 전달되는 클래스를 분석했다.

**ListNode**
```java
public class ListNode {
  int val;
  ListNode next;

  // 기본 생성자
  ListNode () {}
  ListNode (int val) {
    this.val = val;
  }
  ListNode (int val, ListNode next) {
    this.val = val;
    this.next = next;
  }
}
```
ListNode는 단방향 LinkedList로 동작하며, 이전 노드에 대한 정보는 보유하지 않고 있다.
LinkedList의 특징은 다음과 같다.

- 인덱스를 통한 접근이 불가능하다.
- 노드 간의 참조를 보유하여 연결 정보를 구성한다.
- 데이터의 공간을 미리 할당할 필요가 없다.
- 검색 시 연결 정보를 찾는데 시간이 소요된다.
- 단방향으로 구현 시 연결 정보가 끊겨 안정성을 보장할 수 없다.

LinkedList에 대한 이론적인 부분을 찾아보고 문제를 다시 풀어보았다.

### 문제 해결 시 고려사항
문제를 해결하는데 다음과 같은 고려사항들이 존재했다.

- 파라미터들의 연결된 노드 길이가 다를 수 있다.
- 최소 0, 최대 50개의 노드 개수를 가진다.
- 위 조건은 둘 중 하나의 ListNode가 없을 수도 있고, 둘 다 없을 수도 있다.
- 길이가 다르기 때문에 한쪽이 먼저 종료된 이후의 결과를 처리해야한다.

위와 같은 고려사항을 해결하기 위해 다음과 같은 절차로 코드를 작성하기로 했다.

### 코드 작성

#### 의사 코드
```java
class Solution {
  public ListNode mergeTwoLists (ListNode list1, ListNode list2) {
    // 1. 두 개 노드의 길이 확인
    if (노드1 null 여부) {}
    if (노드2 null 여부) {}

    // 2. 결과로 반환할 head 노드 생성
    ListNode 결과;
    // 3. 반복문을 수행하며, 다음 노드를 가리킬 노드 생성
    ListNode 다음노드;

    // 4. 노드1과 노드2가 null 아닐때까지 while 수행
    while (노드1 != null && 노드2 != null) {
      // 5. 대소 비교 후 연결 최신화
      if (노드 1이 더 큰 경우) {}
      else {} // 노드 2가 더 큰 경우

      // 6. 다음 노드 최신화
    }

    // 7. 남은 노드를 추가
    if(노드1,2 == null) {}
  }
}
```
#### 실제 코드
```java
class Solution {
  public ListNode mergeTwoLists (ListNode list1, ListNode list2) {
    // 노드 중 어느 한쪽이 null인 경우 남은 노드 반환
    if(list1 == null) return list2;
    if(list2 == null) return list1;

    // 결과 노드 및 반복 시 다음 노드를 가리키는 노드 생성
    // 여기서 target은 head의 next
    ListNode head = new ListNode();
    ListNode target = head;

    while (list1 != null && list2 != null) {
      if(list1.val > list2.val) {
        target.next = list2;
        list2 = list2.next;
      } else {
        target.next = list1;
        list1 = list1.next;
      }

      // 현재 target을 다음 노드로 갱신
      target = target.next;
    }

    if(list1 != null) {
      target.next = list2;
    } else {
      target.next = list1;
    }

    // head는 기본 생성자로 생성한 노드
    // next가 실제 merge된 노드의 head
    return head.next;
  }
}
```

위의 절차대로 수행되었을 때 흐름을 보자. 파라미터는 다음과 같다.
```java
list1 = [1, 2, 4], list2 = [1, 3, 4]
```
**초기 상태**

![image | 400](https://github.com/user-attachments/assets/dc6aa7f4-425b-4d70-b11f-a2c437c910b2)

**첫 번째 비교**

![image|400](https://github.com/user-attachments/assets/bda2a901-1f5f-4ce2-99c4-05a69f20f9bb)

- `list1.val > list2.val`이 false이므로 target에 list1를 연결
- list1 다음 노드로 갱신
- target 다음 노드로 갱신

**두 번째 비교**

![image|400](https://github.com/user-attachments/assets/e0aebc4e-dca9-41fb-ae2c-3982f30e96fa)

- `list1.val > list2.val`이 true이므로 target에 list2를 연결
- list2 다음 노드로 갱신
- target 다음 노드로 갱신

**세 번째 비교**

![image|400](https://github.com/user-attachments/assets/c072dad6-9d22-48df-af65-e011f4e49ad1)

- `list1.val > list2.val`이 false이므로 target에 list1를 연결
- list1 다음 노드로 갱신
- target 다음 노드로 갱신

**네 번째 비교**

![image|400](https://github.com/user-attachments/assets/2e9dfa38-df5a-4a32-b1d9-36ca37db2fba)

- `list1.val > list2.val`이 true이므로 target에 list2를 연결
- list2 다음 노드로 갱신
- target 다음 노드로 갱신

**다섯 번째 비교**

![image|400](https://github.com/user-attachments/assets/49ddf8d9-7107-4f75-8b32-8f5662efb6ec)

- `list1.val > list2.val`이 false이므로 target에 list1를 연결
- list1 다음 노드로 갱신
- target 다음 노드로 갱신

여기까지 `while`을 통한 반복을 수행한 상태이고, 반복이 종료 후 남은 노드를 확인 후 마지막에 연결한다.

**남은 노드 확인**

![image|500](https://github.com/user-attachments/assets/d1ed95de-7a16-4f3b-953e-d02a383af005)

list2가 null이 아니기 때문에 해당 노드를 마지막에 연결하여 결과를 반환하면 된다.

---

### 정리
이 문제는 LeetCode의 Easy로 분류되는 문제이다. 시간이 오래걸린 이유는 다음과 같다.

- 문제를 정확히 분석하지 못함
- 링크드 리스트에 대한 정확한 이해가 부족
- 조건 비교를 통해 어떤 노드를 연결해야할지 명확한 기준을 생각하는데 시간을 낭비
- 머리로만 문제를 풀려고 했던 부분, 그림으로 표현했으면 훨씬 편하게 풀었을지도 모름

문제를 맞히는 것도 중요하지만 해결하는 과정에서 내가 아직 뭘 모르는지 파악하는게 더 중요하다고 생각했다.
