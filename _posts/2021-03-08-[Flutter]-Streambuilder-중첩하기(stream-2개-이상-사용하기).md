---
title: "[Flutter] Streambuilder 중첩하기(stream 2개 이상 사용하기)"
excerpt: Flutter에서 Streambuilder를 중첩함으로써, 한 위젯 안에서 복수의 firebase 데이터를 이용할 수 있는 방법을 정리했습니다.
categories:  Today's-Study
tags: flutter
toc: true
toc_sticky: true
---

## 1. 작성이유

  - 현재(2020-03-08) flutter를 이용한 프로젝트를 진행중이며, firebase의 데이터를 streambuilder로 불러오는 방식으로 위젯을 만들고 있습니다.  
  - streambuilder에서 가져오고 싶은 컬렉션이 여러 개인 경우, 어떻게 
  두 개 이상의 컬렉션을 한 streambuilder에서 사용할 수 있을까 고민했습니다.  
  - 여러 시도 끝에 streambuilder 중첩을 사용해 해결하는 방법을 찾아내어, 이를 나중에 참고할 수 있게 글로 작성했습니다.  

<br>

## 2. Streambuilder 중첩
  - Streambuilder를 중첩함으로써, 한 위젯을 만들 때 stream을 두 개 이상 사용할 수 있습니다.  
  - 아래는 stream을 두 개(stream1, stream2) 사용하는 예시입니다.  

<br>
  ```
  StreamBuilder<QuerySnapshot>(
      stream: stream1,
      builder: (context, snapshot1){
          return Streambuilder<QuerySnapshot>(
              stream: stream2,
              builder: (context, snapshot2){
                  ... // snapshot1, snapshot2를 이용해 위젯을 작성
              },
          );
      },
  );
  ```
<br>


## 3. 실제 해결해야했던 문제(작업 상황)

  - firebase에 있는 A, B 두 개의 collection을 사용해야 한다.
  - A에는 document가 한 개 있고, B에는 여러 개의 document가 있다.  
  - ListView 위젯을 만들어야 하고, B의 document가 하나씩 요소로 들어가고, 
  각 요소는 모두 A의 document를 사용한다.  

<br>

## 4. 문제 해결 방법(코드)
  - Streambuilder 중첩을 이용해 A, B collection을 사용할 수 있게합니다.
  - buildWidget() 함수에 A, B collection의 document를 넘겨주어 ListView의 요소를 만듭니다.  

  <br>

  1. Streambuilder  

  <br>
  ```
  Streambuilder<QuerySnapshot>(

      // collection A
      stream: Firestore.instance.collection('A').snapshots(),
      builder: (context, snapshot1){
          return StreamBuilder<QuerySnapshot>(

              // collection B
              stream: Firestore.instance.collection('B').snapshots(),
              builder: (context, snapshot2){

                  // A에는 document가 하나만 있기 때문에 first를 사용
                  final aDocument = snapshot1.data.documents.first;
                  final bDocuments = snapshot2.data.documents;

                  return ListView(
                      children: bDocuments.map((doc) => buildWidget(doc, aDocument)).toList(),
                  );
              },
          );
      },
  ),
  ```
  <br>

  1. buildWidget

  <br>
  ```
  Widget buildWidget(DocumentSnapshot doc, DocumentSnapshot doc2){
      ... // ListView의 요소 작성
  }
  ```