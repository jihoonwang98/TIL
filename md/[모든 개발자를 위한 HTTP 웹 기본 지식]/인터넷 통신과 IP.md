### 인터넷 네트워크

- **인터넷 통신**
- **IP(Internet Protocol)**
- **TCP, UDP**
- **PORT**
- **DNS**





## 인터넷 통신

인터넷 망은 **노드**라고 하는 중간 서버들로 구성되어 있다.



![](https://lh4.googleusercontent.com/3ee2hV52cMQEPYT8xWFQ03cHMZ0qwalQTqgF6aW-zwJgzDm5ROXZaGo6cpKkDegENiLO7CVVEjWay90fxMxHVwEfakD6vhWieIFp2R68cN7e-IQHMY8Bu6GHnplmcqxexIquaho_)





## IP(Internet Protocol)



### IP의 역할

- **지정한 IP 주소(IP Address)에 데이터 전달**
- **패킷(Packet)이라는 통신 단위로 데이터 전달**





### IP 패킷 정보

![](https://lh4.googleusercontent.com/09r5iwtb4oHXWrLkWa-Zm-qPJdf18lCdosbg27_8Oqczw26Hrlu8NkJbPhi9Y1_6cGA-_Tujo2UgxsjeySFAPP-bn3HHYzJE4HlcjH9sieyvtjyLbf5J-ldwzkLyajDRZCxh08Tw)





![](https://lh3.googleusercontent.com/5dpOvHvkUli-Y-q1bEtzzU4NcFJ_XnQHwzbzzHNOb3fwXIRJAPFKyOxTakJLsbr8NlzIs5olg9214oyeRT-ntVG5heEZfw4Fn9I55HQQxIhprXHujk4Eg4CPDiIbZGMtqUN3ujWo)



노드끼리 정보를 던져서 최종적으로 서버에 도달함.





![](https://lh6.googleusercontent.com/CIYNKm0raeM49mrD_F3hYZgFhpcVd8-eBeQ_LYwgvagC9DSICYKOnK-l9RnbuE3grV53p9rSPHhr99jGfEiglO49UsUFujhC89xPBmRSEimQT2rhd0FExNbh86XpKjuqFznkIc72)



다시 정보를 노드에 던져서 클라이언트로 전달함.









### IP 프로토콜의 한계

- **비연결성**
  - 패킷을 받을 대상이 없거나 서비스 불능 상태여도 패킷 전송
- **비신뢰성**
  - 중간에 패킷이 사라지면?
  - 패킷이 순서대로 안오면?
- **프로그램 구분**
  - 같은 IP를 사용하는 서버에서 통신하는 애플리케이션이 둘 이상이면?





![](https://lh4.googleusercontent.com/cOJED0zpbbCg2aRlx7-OZ8bf-kzOYL3eXq6xGtBwVuu59GzwwH7NTEfXKK86U_ynstSZ5WnZl8Uve2-0fn99erjV8Pfvf62oc_QTmvq-8l-dXfMeE_5FE29a9OE11SGGCQJpkMoA)



서버가 켜졌는지 꺼졌는지 정보를 받을 수 있는지 없는지를 모르고 그냥 일단 패킷을 보냄







![](https://lh3.googleusercontent.com/sGSddWOR2ld1fKGl1ac_1Mi7odBY4cz7a4yrOp83Q9NbUZFGG9vWJIxQEOd5AWCdg8NP79g4GGrtsQ3trIBTXKNbYZIOcsApmdf42Un8MlP0o5z1-zZ3lr90lRYrwJEwJ-iCDWvv)



패킷을 보냈는데 노드가 패킷을 전달하는 과정에서 확 꺼져버릴 수도 있다...







![](https://lh3.googleusercontent.com/rosULJmPNrpuhMFSjhenLeFtHpZqpsxu2g-8OI-2wlpQwy-cdFnv75jxS3FxivoLwNBtQ4nxlbO2TA7PGtukx5Gnl0AaRhL9_knffYB34TBlJGA7i7ww708rnJ3mxR70Rj6t2ndA)



패킷의 용량이 큰 경우 정보를 여러 개의 패킷에 나눠서 보내는데, 이때 패킷의 전달 순서가 뒤바뀔 수 있다.





이러한 문제들을 어떻게 해결할까??? 

**-> TCP 프로토콜!!**



> 본 포스팅은 인프런의 **모든 개발자를 위한 HTTP 웹 기본 지식 by 김영한** 강좌를 듣고 정리한 내용입니다. 
>
> 링크: https://inf.run/tHdt