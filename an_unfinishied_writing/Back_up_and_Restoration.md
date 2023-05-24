# 백업과 복구
## MySQL Buffer Pool, Redo Log, Log Buffer

![img](https://ssup2.github.io/images/theory_analysis/MySQL_Buffer_Pool_Redo_Log_Log_Buffer/Buffer_Pool_Redo_Log_Log_Buffer.PNG)

### Buffer Pool
InnoDB가 table과 index data를 캐시하는 Memory 영역이다. 자주 사용하는 데이터를 Buffer Pool에 저장하여 Disk가 아닌 Memory에서 직접 접근할 수 있어 DB 성능을 향상시킨다. 

데이터 갱신 시에는 갱신 대상의 데이터를 포함한 페이지가 Buffer Pool에 있는지 확인하고 없으면 Disk의 데이터 파일로부터 읽어들인다. 이후 Buffer Pool에서 데이터를 갱신한다. 

(Disk에 직접 작성하는 것은 임의 장소에 무작위로 액세스해서 쓰기를 수행하기 때문에 성능이 낮아진다.)

### Redo Log
데이터 변경작업을 기록하는 디스크 영역. 데이터 갱신시 커밋과 함께 log에 기록된다. crash 발생 시 데이터 복구에 사용된다. crash가 발생하면 메모리 영역의 Buffer Pool의 데이터는 날라가기 때문에 Redo Log에 저장된 데이터를 바탕으로 장애 발생하기 이전 시점으로(가장 마지막으로 commit된 데이터까지) 복구한다. (InnoDB는 서버 실행 동안에는 Redo Log를 읽지 않는다.)

#### Log Buffer
Disk의 Redo Log file에 작성될 데이터를 저장한 메모리 영역. Log Buffer에 transaction내에서 데이터 변경사항을 저장하고 한꺼번에 Redo Log에 저장한다.
Disk 영역의 Redo Log 접근 횟수를 줄여주는 역할을 한다. InnoDB의 Write, flush 동작으로 Redo Log에 저장된다.

참고 - [Redo Log](https://ssup2.github.io/theory_analysis/MySQL_Buffer_Pool_Redo_Log_Log_Buffer/)

### checkpoint
Buffer Pool에 저장된 데이터를 Disk의 데이터 파일(=데이터베이스 파일)에 반영하는 것을 checkpoint라고 한다. MySQL에서는 flush가 한번에 발생하지 않고 조금씩 Disk로 flush되며 이를 Fuzzy Checkpoint라고 한다.
언제 Fuzzy checkpoint가 발생할까?  여러 경우중 하나는 Redo Log 2개 중 1개가 거의 다 차서 이용할 수 없을 때. 가득찬 Redo Log를 놔두고 다른 Redo Log에 내용을 기록하며, 가득찬 log는 Fuzzy checkpoint가 실행된 후 불필요해져서 비워진다.

#### flush
메모리 영역이나 임시 디스크 저장 영역의 버퍼에 저장된 변경사항들을 데이터베이스 파일에 작성하는 것을 flush라고 한다. 정기적으로 flush되는 것에는 redo, undo log 그리고 buffer pool이 있다.

## DBMS의 3가지 구조
1. REDO LOG(WAL, Write Ahead Log)
2. Buffer Pool(데이터베이스 버퍼)
3. 크래시 복구

Buffer Pool을 사용하여 Disk IO를 줄임으로 성능을 향상시키고 Redo Log로 DB에 예상치 못한 문제 발생시 데이터를 복구하여 지속성을 확보한다. 그러나 물리적 파손이나 논리적 파괴(DDL문으로 테이블 파기)에는 대응할 수 없다.


 

### Q&A
- 레코드란? - 테이블의 row를 다른말로 record라고 한다.

## 참조
- 데이터베이스 첫걸음 - 미크, 기무라 메이지(한빛미디어)
- https://ssup2.github.io/theory_analysis/MySQL_Buffer_Pool_Redo_Log_Log_Buffer/
- https://blog.ex-em.com/1700